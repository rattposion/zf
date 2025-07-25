<template>
  <div v-if="userProfile === 'superadmin'">
    <q-table flat
      bordered
      square
      hide-bottom
      class="my-sticky-dynamic q-ma-lg"
      title="TenantApis"
      :data="tenantApis"
      :columns="columns"
      :loading="loading"
      row-key="id"
      :pagination.sync="pagination"
      :rows-per-page-options="[0]">
      <template v-slot:top-right>
        <q-btn color="primary"
          label="Adicionar"
          @click="tenantApiEdicao = {}; modalTenantApi = true" />
      </template>
      <template v-slot:body-cell-acoes="props">
        <q-td class="text-center">
          <q-btn flat
            round
            icon="edit"
            @click="editarTenantApi(props.row)" />
          <q-btn flat
            round
            icon="mdi-delete"
            @click="deletarTenantApi(props.row)" />
        </q-td>
      </template>
    </q-table>
    <div>
      <q-card class="q-ma-md">
        <q-card-section>
          <q-item-label style="word-break: break-all;">
            <h3 class="text-weight-medium text-nowrap q-pr-md">
              <span class="text-bold">Rota para criação de um novo tenant via API:</span>
            </h3>
            <pre><b>Endpoint:</b> {{ montarUrlIntegração() }}</pre>
            <pre><b>Bearer Token:</b> Api Token</pre>
            <pre><b>Body:</b> </pre>
            <pre>
{
  "status": "active",
  "name": "Empresa Exemplo",
  "maxUsers": 3,
  "maxConnections": 3,
  "acceptTerms": true,
  "email": "user@example.com",
  "password": "securePassword123!",
  "userName": "Pedro Bastos",
  "profile": "admin"
}
            </pre>
          </q-item-label>
        </q-card-section>
      </q-card>
    </div>
    <div>
      <q-card class="q-ma-md">
        <q-card-section>
          <q-item-label style="word-break: break-all;">
            <h3 class="text-weight-medium text-nowrap q-pr-md">
              <span class="text-bold">Rota para ativação/desativação de um novo tenant via API:</span>
            </h3>
            <pre><b>Endpoint:</b> {{ montarUrlIntegraçãoStatus() }}</pre>
            <pre><b>Bearer Token:</b> Api Token</pre>
            <pre><b>Body:</b> </pre>
            <pre>
{
  "status": "active",
  "identity": "07122989674"
}
            </pre>
          </q-item-label>
        </q-card-section>
      </q-card>
    </div>
    <ModalTenantApi :modalTenantApi.sync="modalTenantApi"
      :tenantApiEdicao.sync="tenantApiEdicao"
      @modal-tenantApi:criada="tenantApiCriada"
      @modal-tenantApi:editada="tenantApiEditada" />
  </div>
</template>

<script>
import { DeletarApiTenant, ListarApisTenant } from 'src/service/tenantApi'
import ModalTenantApi from './ModalTenantApi'
export default {
  name: 'TenantApis',
  components: {
    ModalTenantApi
  },
  data () {
    return {
      userProfile: 'user',
      tenantApiEdicao: {},
      modalTenantApi: false,
      tenantApis: [],
      pagination: {
        rowsPerPage: 40,
        rowsNumber: 0,
        lastIndex: 0
      },
      loading: false,
      columns: [
        { name: 'id', label: '#', field: 'id', align: 'left' },
        { name: 'apiToken', label: 'Api Token', field: 'apiToken', align: 'left' },
        { name: 'acoes', label: 'Ações', field: 'acoes', align: 'center' }
      ]
    }
  },
  computed: {
    cBaseUrlIntegração () {
      return `${process.env.URL_API}/tenantApiStoreTenant`
    },
    cBaseUrlIntegraçãoStatus () {
      return `${process.env.URL_API}/tenantApiUpdateTenant`
    }
  },
  methods: {
    montarUrlIntegração (id) {
      return `${this.cBaseUrlIntegração}`
    },
    montarUrlIntegraçãoStatus (id) {
      return `${this.cBaseUrlIntegraçãoStatus}`
    },
    async listarTenantApis () {
      const { data } = await ListarApisTenant()
      this.tenantApis = data.tenantApi
    },
    tenantApiCriada (tenantApi) {
      const newTenantApis = [...this.tenantApis]
      newTenantApis.push(tenantApi)
      this.tenantApis = [...newTenantApis]
    },
    tenantApiEditada (tenantApi) {
      const newTenantApis = [...this.tenantApis]
      const idx = newTenantApis.findIndex(f => f.id === tenantApi.id)
      if (idx > -1) {
        newTenantApis[idx] = tenantApi
      }
      this.tenantApis = [...newTenantApis]
    },
    editarTenantApi (tenantApi) {
      this.tenantApiEdicao = { ...tenantApi }
      this.modalTenantApi = true
    },
    deletarTenantApi (tenantApi) {
      this.$q.dialog({
        title: 'Atenção!!',
        message: `Deseja realmente deletar a TenantApi "${tenantApi.id}"?`,
        cancel: {
          label: 'Não',
          color: 'primary',
          push: true
        },
        ok: {
          label: 'Sim',
          color: 'negative',
          push: true
        },
        persistent: true
      }).onOk(() => {
        this.loading = true
        DeletarApiTenant(tenantApi)
          .then(res => {
            let newTenantApis = [...this.tenantApis]
            newTenantApis = newTenantApis.filter(f => f.id !== tenantApi.id)

            this.tenantApis = [...newTenantApis]
            this.$q.notify({
              type: 'positive',
              progress: true,
              position: 'top',
              message: `TenantApi ${tenantApi.apiToken} deletada!`,
              actions: [{
                icon: 'close',
                round: true,
                color: 'white'
              }]
            })
          })
        this.loading = false
      })
    }

  },
  mounted () {
    this.listarTenantApis()
    this.userProfile = localStorage.getItem('profile')
  }
}
</script>

<style lang="scss" scoped>
</style>
