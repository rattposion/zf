<template>
  <div v-if="userProfile === 'admin' || userProfile === 'super' ">
    <q-table
      flat
      bordered
      square
      hide-bottom
      class="my-sticky-dynamic q-ma-lg"
      :data="agendadas"
      :columns="columns"
      :loading="loading"
      row-key="id"
      :pagination.sync="pagination"
      :rows-per-page-options="[0]"
    >
      <template v-slot:top>
        <div class="q-pa-md" style="display: flex; justify-content: space-between; align-items: center; gap: 20px;">
          <div>
            <div class="text-h6">Agendamentos</div>
          </div>
          <div>
            <q-btn @click="onClickOpenAgendamentoMensagem()"
              flat
              dense
              icon="mdi-alarm-plus"
              color="primary"
              class="bg-padrao btn-rounded">
              <q-tooltip content-class="bg-primary text-bold">
                Adicionar Agendamento
              </q-tooltip>
            </q-btn>
          </div>
        </div>
      </template>


      <template v-slot:body-cell-color="props">
        <q-td class="text-center">
            {{ props.row.contact.name }}
        </q-td>
      </template>
      <template v-slot:body-cell-acoes="props">
        <q-td class="text-center">
          <q-btn
            flat
            round
            icon="mdi-delete"
            @click="deletarMensagem(props.row)"
          />
        </q-td>
      </template>
    </q-table>
  </div>
</template>



<script>
import { ListarAgendadas, DeletarMensagem } from 'src/service/tickets'
// import mixinCommon from './mixinCommon'
export default {
  name: 'Agendamentos',
  // mixins: [mixinCommon],
  data () {
    return {
      userProfile: 'user',
      agendadas: [],
      pagination: {
        rowsPerPage: 40,
        rowsNumber: 0,
        lastIndex: 0
      },
      loading: false,
      columns: [
        { name: 'id', label: '#', field: 'id', align: 'left' },
        { name: 'body', label: 'Mensagem', field: row => row.body.substring(0, 20) + '...', align: 'left' },
        { name: 'ticketId', label: 'Ticket', field: 'ticketId', align: 'center' },
        { name: 'ticketStatus', label: 'Ticket', field: row => this.formatTicketStatus(row.ticketStatus), align: 'center' },
        { name: 'mediaType', label: 'Tipo', field: row => this.formatMessageType(row.mediaType), align: 'center' },
        { name: 'contactName', label: 'Contato', field: 'contactName', align: 'center' },
        { name: 'scheduleDate', label: 'Data', field: 'scheduleDate', align: 'center' },
        { name: 'acoes', label: 'Ações', field: 'acoes', align: 'center' }
      ]
    }
  },
  methods: {
    async listarAgendadas () {
      try {
        this.loading = true;
        const data = await ListarAgendadas();
        console.log(data.data)
        this.agendadas = data.data.messages
        .filter(message => !message.isDeleted)
        .map(message => ({
          ...message,
          contactName: message.contact.name,
          ticketId: message.ticket.id,
          ticketStatus: message.ticket.status
        }));
      } catch (error) {
        console.error('Erro ao listar as mensagens:', error);
      } finally {
        this.loading = false;
      }
    },
    formatTicketStatus(status) {
      switch (status) {
        case 'open':
          return 'Aberto';
        case 'closed':
          return 'Fechado';
        case 'pending':
          return 'Pendente';
        case 'schedule':
          return 'Agendado';
        default:
          return status;
      }
    },
    formatMessageType(type) {
      switch (type) {
        case 'chat':
          return 'Texto';
        case 'image':
          return 'Imagem';
        case 'templates':
          return 'Template';
        case 'video':
          return 'Vídeo';
        case 'application':
          return 'Arquivo';
        case 'text':
          return 'Arquivo';
        default:
          return type;
      }
    },
    deletarMensagem(mensagem) {
      this.$q
        .dialog({
          title: 'Atenção!! Deseja realmente deletar essa mensagem? ',
          message: 'Mensagens antigas não serão apagadas no cliente.',
          cancel: {
            label: 'Não',
            color: 'primary',
            push: true,
          },
          ok: {
            label: 'Sim',
            color: 'negative',
            push: true,
          },
          persistent: true,
        })
        .onOk(() => {
          this.loading = true
          DeletarMensagem(mensagem)
            .then((res) => {
              this.loading = false
              mensagem.isDeleted = true
              window.location.reload();
            })
            .catch((error) => {
              this.loading = false
              console.error(error)
              this.$notificarErro('Não foi possível apagar a mensagem', error)
            })
        })
        .onCancel(() => {})
    },
    onClickOpenAgendamentoMensagem() {
      const dialog = this.$q.dialog({
        component: () => import('./DialogAgendamentoMensagem.vue'),
        parent: this
      })
    },
  },
  mounted () {
    this.listarAgendadas()
    this.userProfile = localStorage.getItem('profile')
  }
}
</script>

<style lang="scss" scoped>
</style>
