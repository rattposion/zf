<template>
  <div id="app" class="q-pa-sm">
    <div class="q-my-md">
      <q-card
        flat
        bordered
        class="full-width"
      >
        <q-card-section class="text-h6 text-bold">
          Controle Visual de Atendimentos
        </q-card-section>
        <div v-if="loading">Aguarde, carregando as informações do Kanban...</div>
        <div v-if="loading" class="loading-bar">
          <div class="bar"></div>
        </div>
      </q-card>
    </div>
    <q-card class="q-my-md"> 
      <div class="q-pa-md">
        <drag-drop
          :dropzones="dropGroups"
          :dropzonesTitle="'Atendimentos'"
          :originalData="stories"
          :originalTitle="'Contatos'"
          :inPlace="true"
          :enableSave="true"
          :enableCancel="true"
          @save="save"
          @cancel="cancel"
        >
          <template #dd-card="{ cardData }">
            <!-- <cards 
              :data="cardData" 
              @done="doneMarked" 
            /> -->
            <cards 
              :data="cardData" 
              @abrirTicket="handleSaveTicket(cardData, 'whatsapp')" 
              @done="doneMarked"
            />
          </template>
        </drag-drop>
        <!-- <drag-drop
          :dropzones="dropGroups"
          :dropzonesTitle="'Atendimentos'"
          :originalData="stories"
          :originalTitle="'Contatos'"
          :inPlace="true"
          :enableSave="true"
          :enableCancel="true"
          @save="save"
          @cancel="cancel"
        >
          <template #dd-card="{ cardData }">
            <cards 
              :data="cardData" 
              @done="doneMarked" 
            />
            <cards 
              :data="cardData" 
              @abrirTicket="handleSaveTicket(cardData, 'whatsapp')" 
            />
          </template>
        </drag-drop> -->
      </div>
    </q-card>
  </div>
</template>

<script>
const userId = +localStorage.getItem('userId')
import DragDrop from 'vue-drag-n-drop'
import Cards from './Cards.vue'
import { ListarContatosKanban, EditarContato } from 'src/service/contatos'
import { ListarKanbans } from 'src/service/kanban'
import { CriarTicket } from 'src/service/tickets'
import DefaultProfile from '../../assets/default-profile.png'
import { mapGetters } from 'vuex'

export default {
  components: {
    DragDrop,
    Cards
  },
  data() {
    return {
      dropGroups: [],
      stories: [],
      loading: false,
      params: {
        pageNumber: 1,
        searchParam: null,
        hasMore: true
      },
    }
  },
  computed: {
    ...mapGetters(['whatsapps'])
  },
  methods: {
    async doneMarked(item) {
      const contato = {
        number: item.description,
        kanban: null
      }

      this.$q.dialog({
        title: `Remover atendimento do Painel`,
        message: 'Deseja remover o atendimento?',
        ok: {
          push: true,
          color: 'positive',
          label: 'Iniciar'
        },
        cancel: {
          push: true,
          label: 'Cancelar',
          color: 'negative'
        },
        persistent: true
      }).onOk(async () => {
        try {
          if (item.id) {
            const { data } = await EditarContato(item.id, contato)
            console.log(data)
            this.$q.notify({
              type: 'info',
              progress: true,
              position: 'top',
              textColor: 'black',
              message: 'Contato editado!',
              actions: [{
                icon: 'close',
                round: true,
                color: 'white'
              }]
            })
          }
          window.location.reload();
        } catch (error) {
          console.error(error)
          this.$notificarErro('Ocorreu um erro ao remover o contato', error)
        }
      })
    },
    async handleSaveTicket (card, channel) {
      if (!card.id) return
      
      const itens = []
      const channelId = null
      this.whatsapps.forEach(w => {
        if (w.type === channel) {
          itens.push({ label: w.name, value: w.id })
        }
      })

      this.$q.dialog({
        title: `Contato: ${card.title}`,
        message: 'Selecione o canal para iniciar o atendimento.',
        options: {
          type: 'radio',
          model: channelId,
          // inline: true
          isValid: v => !!v,
          items: itens
        },
        ok: {
          push: true,
          color: 'positive',
          label: 'Iniciar'
        },
        cancel: {
          push: true,
          label: 'Cancelar',
          color: 'negative'
        },
        persistent: true
      }).onOk(async channelId => {
        if (!channelId) return
        // this.loading = true
        try {
          console.log('ok')
          const { data: ticket } = await CriarTicket({
            contactId: card.id,
            isActiveDemand: true,
            userId: userId,
            channel,
            channelId,
            status: 'open'
          })
          console.log('ok 2')
          // await this.$store.commit('SET_HAS_MORE', true)
          await this.$store.dispatch('AbrirChatMensagens', ticket)
          this.$q.notify({
            message: `Atendimento Iniciado || ${ticket.contact.name} - Ticket: ${ticket.id}`,
            type: 'positive',
            position: 'top',
            progress: true,
            actions: [{
              icon: 'close',
              round: true,
              color: 'white'
            }]
          })
          this.$router.push({ name: 'chat', params: { ticketId: ticket.id } })
        } catch (error) {
          if (error.status === 409) {
            const ticketAtual = JSON.parse(error.data.error)
            this.abrirAtendimentoExistente(card, ticketAtual)
            return
          }
          this.$notificarErro('Ocorreu um erro!', error)
        }
        // this.loading = false
      })
    },
    abrirAtendimentoExistente (card, ticket) {
      this.$q.dialog({
        title: 'Atenção!!',
        message: `${card.title} possui um atendimento em curso (Atendimento: ${ticket.id}). Deseja abrir o atendimento?`,
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
      }).onOk(async () => {
        try {
          this.abrirChatContato(ticket)
        } catch (error) {
          this.$notificarErro(
            'Não foi possível atualizar o token',
            error
          )
        }
      })
    },
    abrirChatContato (ticket) {
      if (this.$q.screen.lt.md && ticket.status !== 'pending') {
        this.$root.$emit('infor-cabecalo-chat:acao-menu')
      }
      if (!(ticket.status !== 'pending' && (ticket.id !== this.$store.getters.ticketFocado.id || this.$route.name !== 'chat'))) return
      this.$store.commit('SET_HAS_MORE', true)
      this.$store.dispatch('AbrirChatMensagens', ticket)
    },
    async save(data) {
      this.$q.notify({
        type: 'warning',
        progress: true,
        position: 'top',
        textColor: 'black',
        message: 'Carregando dados novos no Kanban... Aguarde!',
        actions: [{
          icon: 'close',
          round: true,
          color: 'white'
        }]
      });

      const contatos = [];

      if (data.originalBucket) {
        for (const bucket of data.originalBucket) {
          const contato = {
            number: bucket.description,
            kanban: null
          };
          contatos.push({ id: bucket.id, contato });
        }
      }

      if (data.dropzones) {
        for (const dropzone of data.dropzones) {
          for (const child of dropzone.children) {
            const contato = {
              number: child.description,
              kanban: dropzone.id
            };
            contatos.push({ id: child.id, contato });
          }
        }
      }

      try {
        const promessas = contatos.map(async (contatoInfo) => {
          if (contatoInfo.id) {
            const { data } = await EditarContato(contatoInfo.id, contatoInfo.contato);
            console.log(data);
          }
        });

        await Promise.all(promessas);

        this.$q.notify({
          type: 'info',
          progress: true,
          position: 'top',
          textColor: 'black',
          message: 'Todos os cards foram editados com sucesso!',
          actions: [{
            icon: 'close',
            round: true,
            color: 'white'
          }]
        });
      } catch (error) {
        console.error(error);
        this.$notificarErro('Ocorreu um erro ao editar os contatos', error);
      }
    },
    cancel() {
      window.location.reload()
    },
    async listarContatos () {
      try {
        this.loading = true;
        while (this.params.hasMore) {
          const { data } = await ListarContatosKanban(this.params)
          data.contacts.sort((a, b) => a.id - b.id);
          if (data.contacts && Array.isArray(data.contacts)) {
            data.contacts.forEach(async (contato) => {
              if(!contato.isGroup || contato.kanban !== null){
                const newContact = {
                  title: contato.name || contato.pushname || 'Nome não disponível',
                  description: contato.number,
                  id: contato.id,
                  profilePicUrl: contato.profilePicUrl || DefaultProfile,
                  tags: contato.tags,
                  wallet: contato.wallet,
                  done: false
                }
                if (contato.kanban === null) {
                  return null
                } else {
                  console.log(contato)
                  const kanbanId = contato.kanban
                  const kanbanLane = this.dropGroups.find(group => group.id === kanbanId)
                  if (kanbanLane) {
                    kanbanLane.children.push(newContact)
                  }
                  else{
                    return null
                  }
                }
              }
            })
          }
          this.params.hasMore = data.hasMore;
          this.params.pageNumber++;
        }
      } catch (error) {
        console.error('Erro ao obter contatos:', error)
      } finally {
        this.loading = false;
      }
    },
    async listarKanbans () {
      const { data } = await ListarKanbans()
      this.kanbans = data.kanban
      this.kanbans.sort((a, b) => a.id - b.id)
      this.kanbans.forEach(kanban => {
        this.dropGroups.push({
          id: kanban.id,
          name: kanban.name,
          children: []
        })
      })
    },
  },
  mounted () {
    this.listarKanbans()
    this.listarContatos()
  }
}
</script>

<style lang="sass">
#app
  text-align: center
  color: $primary
  margin-top: 60px

  h2
    color: $primary

  dd-card
    display: inline-block
    vertical-align: top
    margin: 10px

.dd-drop-container
  color: $primary
  width: 320px !important

.q-pa-md
  margin: 50px 0

.button.dd-save
  background-color: $primary !important
  color: #FFF !important
  
.button.dd-cancel
  background-color: $primary !important
  color: #FFF !important

.dd-title, .dd-first-group, hr
    display: none

.loading-bar
  width: 100%
  height: 4px
  background-color: #ccc
  position: relative
  
  .bar
    width: 0
    height: 100%
    background-color: #007bff
    position: absolute
    top: 0
    left: 0
    animation: loadingAnimation 1s infinite
  
@keyframes loadingAnimation
  0%
    width: 0
  100%
    width: 100%
  
</style>
<!-- .dd-drop-actions .dd-save::before
  content: "Salvar"

.dd-drop-actions .dd-cancel::before
  content: "Cancelado" -->