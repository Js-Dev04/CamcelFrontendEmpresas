<template>
  <div>
    <valid-delete-enterprise-menu
      v-if="showDeleteMenu"
      :show="showDeleteMenu"
      @handleDeleteMenuClose="handleDeleteMenuClose"
      @handleDeleteMenuAccept="handleDeleteMenuAccept"
    />

    <div v-if="!isLoading" class="q-mx-auto" style="max-width: 1000px">
      <q-img
        :src="`${api_base_backend}/${enterprise.image}`"
        alt="esta enterprise no pose imagen"
        style="height: 350px"
        :fit="cover"
      >
        <template v-slot:error>
          <div class="absolute-full text-subtitle2 flex flex-center">
            <h4 class="text-h4">
              {{ enterprise.name }}
            </h4>
          </div>
        </template>
        <div class="absolute-full text-subtitle2 flex flex-center">
          <h4 class="text-h4">
            {{ enterprise.name }}
          </h4>
        </div>
      </q-img>
      <div class="row justify-between">
        <div class="text-caption">
          <div class="text-grey row items-center">
            {{ enterprise.is_valid ? "Verificado" : "No verificado" }}
            <q-icon
              :name="enterprise.is_valid ? 'check_circle' : 'cancel'"
              :color="enterprise.is_valid ? 'green' : 'red'"
              size="30px"
            />
            <q-card-section class="q-pt-none q-px-none q-ml-sm">
              <div class="flex justify-left items-center q-mt-md w-full">
                <q-btn
                  label="Editar"
                  type="button"
                  class="q-mr-sm"
                  color="primary"
                  @click="enterpriseEditMenu = true"
                />
                <edit-enterprise
                  v-if="enterpriseEditMenu"
                  :enterprise="enterprise"
                  :show="enterpriseEditMenu"
                  @handleCloseMenuEditEnterprise="handleCloseMenuEditEnterprise"
                />

                <q-btn
                  v-if="enterprise.is_valid"
                  label="Desvalidar"
                  type="button"
                  color="negative"
                  @click="handleNotValidEnterprise"
                />

                <q-btn
                  v-else
                  label="Validar"
                  type="button"
                  color="secondary"
                  @click="handleValidEnterprise"
                />

                <q-btn
                  type="button"
                  class="text-lg text-negative q-ml-md"
                  @click="()=>{showDeleteMenu=true}"
                >
                  Eliminar <span class="mdi mdi-trash-can"></span>
                </q-btn>
              </div>
            </q-card-section>
          </div>
        </div>
        <div style="width: 400px" v-if="enterprise.user">
          <h5 class="text-h5 q-my-none">Encargado de la enterprise</h5>
          <q-card>
            <q-card-section>
              Nombre: {{ enterprise.user.name }}
            </q-card-section>
            <q-card-section>
              Email: {{ enterprise.user.email }}
            </q-card-section>
          </q-card>
        </div>
      </div>
      <operators-list :enterprise="enterprise.slug" />
      <table-documents :documents="documents"/>
    </div>

    <div v-if="enterpriseNoExiste" class="text-center">Cargando...</div>
    <div v-if="isLoading" class="row justify-center">
        <q-card style="width: 900px">
      <q-card-section>
        <div class="text-h6">Crear Empresa</div>
      </q-card-section>

      <q-card-section class="q-pt-none">
        <q-form @submit.prevent="handleCreateEnterprise">
          <q-input
            name="RUT"
            required
            label="RUT"
            v-model="dataCreateEnterprise.RUT"
          />
          <div
            v-for="(error, index) in error_create?.RUT"
            :key="index"
            class="q-mt-sm"
          >
            <span  class="q-pa-xs bg-negative text-white">
              {{ error }}
            </span>
          </div>
          <q-input
            name="nombre"
            required
            label="nombre"
            v-model="dataCreateEnterprise.nombre"
          />
          <div
            v-for="(error, index) in error_create?.nombre"
            :key="index"
            class="q-mt-sm"
          >
            <span class="q-pa-xs bg-negative text-white">{{ error }}</span>
          </div>
          <q-checkbox
            label="Verificado"
            v-model="dataCreateEnterprise.is_valid"
          />

          <q-file
            color="teal"
            filled
            label="image"
            v-model="dataCreateEnterprise.image"
          >
            <template v-slot:prepend>
              <q-icon name="cloud_upload" />
            </template>
          </q-file>
          <div
            v-for="(error, index) in error_create?.image"
            :key="index"
            class="q-mt-sm"
          >
            <span class="q-pa-xs bg-negative text-white">{{ error }}</span>
          </div>
          <p v-if="isLoadingUser">loading...</p>
          <q-select
            v-else
            v-model="dataCreateEnterprise.user_id"
            required
            option-label="email"
            :options="users"
            label="Usuario Empresario"
          />

          <div
            v-for="(error, index) in error_create?.user_id"
            :key="index"
            class="q-mt-sm"
          >
            <span class="q-pa-xs bg-negative text-white">{{ error }}</span>
          </div>
          <q-btn label="Crear" class="q-mt-md" type="submit" color="primary" />
        </q-form>
      </q-card-section>

      <q-card-actions align="right">
        <q-btn
          flat
          label="Cerrar"
          color="primary"
          v-close-popup
          @click="handleCloseCreateEnterprise"
        />
      </q-card-actions>

    </q-card>
    </div>
  </div>
</template>
<script>
import EditEnterprise from "src/components/enterprises/EditEnterprise.vue";
import OperatorsList from "src/components/operators/OperatorsList.vue";
import MenuCreateOperator from "src/components/MenuCreateOperator.vue";
import MenuCreateEmpresa from "src/components/CreateEmpresa.vue"
import { useRoute, useRouter } from "vue-router";
import { ref } from "vue";
import { api_base_backend } from "../helpers.js";
import TableDocuments from "../components/documents/TableDocuments.vue";
import {
  useEnterprise,
  useValidEnterprise,
  useNotValidEnterprise,
  useDeleteEnterprise,
} from "src/hooks/api/enterprises.hooks";
import { useDocuments } from "src/hooks/api/documents.hooks";
import ValidDeleteEnterpriseMenu from "src/components/ValidDeleteMenu.vue";
import { reactive, toRef } from "vue";
import { api } from "src/boot/axios";
import { useEnterpriseStore } from "src/store/enterprise.store";

export default {
  props: {
    show: {
      type: Boolean,
      required: true,
    }, },
  components: {
    EditEnterprise,
    MenuCreateOperator,
    TableDocuments,
    OperatorsList,
    ValidDeleteEnterpriseMenu,
    MenuCreateEmpresa
  },
  setup (props, { emit }) {
    const enterpriseStore = useEnterpriseStore();

    const isLoadingUser = ref(true);
    const users = ref(null);

    api.get("users", {
      params: {
        rol: "users_enterprise"
      }
    }).then((response) => {
      isLoadingUser.value = false;
      users.value = response.data.users;
    });

    const show = toRef(props, "show");

    const dataCreateEnterprise = reactive({
      nombre: "",
      RUT: "",
      is_valid: false,
      image: null,
      user_id: null,
    });

    const error_create = ref(null);

    const handleCloseCreateEnterprise = () => {
      emit("handleCloseCreateEnterprise");
    };

    const handleCreateEnterprise = () => {
      api
        .post(
          "enterprises",
          {
            ...dataCreateEnterprise,
            user_id: dataCreateEnterprise.user_id?.id,
          },
          {
            headers: dataCreateEnterprise.image
              ? { "Content-Type": "multipart/form-data" }
              : { "Content-Type": "application/json" },
          }
        )
        .then((response) => {
          enterpriseStore.addEnterprise(response.data.enterprise);
          handleCloseCreateEnterprise();
        })
        .catch((err) => {
          if (err.response.status === 422) {
            const messages = err.response.data.errors;
            error_create.value = messages;
          }
        });
    };
    const router = useRouter();
    const route = useRoute();
    const { params } = route;
    const { enterprise, isLoading, refetch } = useEnterprise(params.slug);
    const { documents, isLoading: isLoadingDocuments, refetch: refetchDocuments } = useDocuments(params.slug);

    const enterpriseNoExiste = ref(false);
    const enterpriseEditMenu = ref(false);

    const showDeleteMenu = ref(false);

    const handleDeleteMenuClose = () => showDeleteMenu.value = false;

    const handleDeleteMenuAccept = async () => {
      showDeleteMenu.value = false;
      await useDeleteEnterprise(params.slug);
      router.push({ name: "enterprises" });
    };

    const handleCloseMenuEditEnterprise = () => {
      enterpriseEditMenu.value = false;
      refetch();
    };

    const handleNotValidEnterprise = async () => {
      await useNotValidEnterprise(params.slug);
      refetch();
    };

    const handleValidEnterprise = async () => {
      await useValidEnterprise(params.slug);
      refetch();
    };

    return {
      isLoading: isLoading  && isLoadingDocuments,
      enterprise,
      documents,
      isLoadingDocuments,
      refetchDocuments,
      showDeleteMenu,
      enterpriseNoExiste,
      handleDeleteMenuClose,
      handleDeleteMenuAccept,
      api_base_backend,
      enterpriseEditMenu,
      handleCloseMenuEditEnterprise,
      handleNotValidEnterprise,
      handleValidEnterprise,
      dataCreateEnterprise,
      handleCreateEnterprise,
      show,
      handleCloseCreateEnterprise,
      isLoadingUser,
      users,
      error_create,
    };
  },
}
</script>