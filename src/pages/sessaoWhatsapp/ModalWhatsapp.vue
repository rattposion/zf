<template>
  <q-dialog :value="modalWhatsapp"
    @hide="fecharModal"
    @show="abrirModal"
    persistent>
    <q-card class="q-pa-md"
      style="width: 500px">
      <q-card-section>
        <div class="text-h6">
          <q-icon size="50px"
            class="q-mr-md"
            :name="whatsapp.type ? `img:${whatsapp.type}-logo.png` : 'mdi-alert'" /> {{ whatsapp.id ? 'Editar' :
              'Adicionar'
            }}
          Canal
        </div>
      </q-card-section>
      <q-card-section>
        <div class="row">
          <div class="col-12 q-my-sm">
            <q-select :disable="!!whatsapp.id"
              v-model="whatsapp.type"
              :options="optionsWhatsappsTypes"
              label="Tipo"
              emit-value
              map-options
              filled />
          </div>
          <div class="col-12 q-my-sm" v-if="whatsapp.type === 'whatsapp' && whatsapp.status !== 'CONNECTED'">
            <q-checkbox v-model="showPairingCode" label="Usar Pairing Code" />
          </div>
          <div class="col-12 q-my-sm" v-if="showPairingCode && whatsapp.type === 'whatsapp'">
            <q-input v-model="whatsapp.wppUser" type="number" label="Insira a Conta do WhatsApp" filled />
          </div>
          <!-- <div class="col-12 q-my-sm" v-if="whatsapp.type === 'whatsapp' || whatsapp.type === 'instagram'">
            <q-checkbox v-model="showProxy" label="Usar Proxy" />
          </div>
          <div class="col-12 q-my-sm" v-if="showProxy && (whatsapp.type === 'whatsapp' || whatsapp.type === 'instagram')">
            <q-input v-model="whatsapp.proxyUrl" type="text" label="Insira um Proxy" filled />
          </div> -->
          <div class="col-12"
            style="margin-bottom: 20px; margin-top: 10px;"
            v-if="whatsapp.type === 'whatsapp'">
            <div class="q-tabs row items-center">
              <div class="q-tab" :class="{ 'q-tab--active': activeTab === 'tab1' }" @click="activeTab = 'tab1'" :style="{ backgroundColor: $q.dark.isActive ? '#363636' : '#f2f2f2', borderRadius: '10px 10px 0 0', textTransform: 'capitalize' }">Informações</div>
              <div class="q-tab" :class="{ 'q-tab--active': activeTab === 'tab2' }" @click="activeTab = 'tab2'" :style="{ backgroundColor: $q.dark.isActive ? '#464646' : '#e2e2e2', borderRadius: '10px 10px 0 0', textTransform: 'capitalize' }">Recomendações</div>
            </div>
            
            <div v-if="activeTab === 'tab1'" class="q-pa-md" :style="{ backgroundColor: $q.dark.isActive ? '#363636' : '#f2f2f2' }">
              <li>Antes de conectar seu WhatsApp no sistema, remova a conexão com o WhatsApp Web.</li>
              <li>Depende do smartphone.</li>
              <li>Necessita de reconexão periódica.</li>
            </div>
            
            <div v-if="activeTab === 'tab2'" class="q-pa-md" :style="{ backgroundColor: $q.dark.isActive ? '#464646' : '#e2e2e2' }">
              <li>Para o perfeito funcionamento, mantenha o telefone sincronizado sempre ligado e conectado a internet, desative o descanso de tela e mantenha o app do WhatsApp aberto</li>
              <li>Arquive pelo celular conversas dispensáveis ou mais antigas</li>
              <li>Recomendamos o uso do navegador Chrome e do sistema operacional Windows</li>
              <li>Não abrir o WhatsApp Web com o número sincronizado na plataforma em outro navegador. Caso isso aconteça, o funcionamento poderá ser impactado</li>
              <!-- Conteúdo da segunda aba aqui -->
            </div>
          </div>
          <div class="col-12"
            style="margin-bottom: 20px; margin-top: 10px;"
            v-if="whatsapp.type === 'instagram'">
            <div class="q-tabs row items-center">
              <div class="q-tab" :class="{ 'q-tab--active': activeTab === 'tab1' }" @click="activeTab = 'tab1'" :style="{ backgroundColor: $q.dark.isActive ? '#363636' : '#f2f2f2', borderRadius: '10px 10px 0 0', textTransform: 'capitalize' }">Informações</div>
              <div class="q-tab" :class="{ 'q-tab--active': activeTab === 'tab2' }" @click="activeTab = 'tab2'" :style="{ backgroundColor: $q.dark.isActive ? '#464646' : '#e2e2e2', borderRadius: '10px 10px 0 0', textTransform: 'capitalize' }">Recomendações</div>
            </div>
            
            <div v-if="activeTab === 'tab1'" class="q-pa-md" :style="{ backgroundColor: $q.dark.isActive ? '#363636' : '#f2f2f2' }">
              <li>Serviço Não Oficial.</li>
              <li>Necessita de reconexão periódica.</li>
            </div>
            
            <div v-if="activeTab === 'tab2'" class="q-pa-md" :style="{ backgroundColor: $q.dark.isActive ? '#464646' : '#e2e2e2' }">
              <li>Use preferencialmente VPSs localizadas no Brasil e no seu Estado</li>
              <li>Recomendamos o uso do navegador Chrome e do sistema operacional Windows</li>
            </div>
          </div>
          <div class="col-12"
            style="margin-bottom: 20px; margin-top: 10px;"
            v-if="whatsapp.type === 'waba'">
            <div class="q-tabs row items-center">
              <div class="q-tab" :class="{ 'q-tab--active': activeTab === 'tab1' }" @click="activeTab = 'tab1'" style="background-color: #f2f2f2; border-radius: 10px 10px 0 0; text-transform: capitalize;">Informações</div>
              
            </div>
            
            <div v-if="activeTab === 'tab1'" class="q-pa-md" style="background-color: #f2f2f2; border-radius: 0 0 10px 10px;">
              <li>Estabilidade de conexão garantida</li>
              <li>Sem risco de banimento</li>
              <li>Segurança contra roubo de conta</li>
              <li>Permite o uso de botões</li>
              <li>Permite o uso de templates</li>
            </div>
            
          </div>
          <div class="col-12">
            <c-input outlined
              label="Nome"
              v-model="whatsapp.name"
              :validator="$v.whatsapp.name"
              @blur="$v.whatsapp.name.$touch" />
          </div>
          <!-- <div class="col-12" v-if="whatsapp.type === 'whatsapp'">
            <c-input outlined
              label="Usuário"
              v-model="whatsapp.wppUser"
            />
          </div>
          <div class="col-12" v-if="whatsapp.type === 'whatsapp'">
            <c-input outlined
              label="Senha"
              v-model="whatsapp.wppPass"
            />
          </div> -->
          <template v-if="whatsapp.type === 'messenger'">
            <VFacebookLogin :app-id="cFbAppId"
              @sdk-init="handleSdkInit"
              @login="fbLogin"
              :login-options="FBLoginOptions"
              version="v12.0" />
          </template>
          <div class="col-12 q-mt-md"
            v-if="whatsapp.type === 'telegram'">
            <c-input outlined class="blur-effect"
              label="Token Telegram"
              v-model="whatsapp.tokenTelegram" />
          </div>
          <div class="col-12 q-mt-md"
            v-if="whatsapp.type === 'waba'">
            <c-input outlined class="blur-effect"
              label="Identificação do número de telefone"
              v-model="whatsapp.tokenAPI" />
          </div>
          <div class="col-12 q-mt-md"
            v-if="whatsapp.type === 'waba'">
            <c-input outlined class="blur-effect"
              label="Identificação da conta do WhatsApp Business"
              v-model="whatsapp.wabaId" />
          </div>
          <div class="col-12 q-mt-md"
            v-if="whatsapp.type === 'waba'">
            <c-input outlined class="blur-effect"
              label="Token do Gerenciador de Negócios Meta"
              v-model="whatsapp.bmToken" />
          </div>
          <div class="q-mt-md row full-width justify-center"
            v-if="whatsapp.type === 'instagram'">
            <div class="col">
              <fieldset class="full-width q-pa-md">
                <legend>Dados da conta do Instagram</legend>
                <div class="col-12 q-mb-md"
                  v-if="whatsapp.type === 'instagram'">
                  <c-input outlined
                    label="Usuário"
                    v-model="whatsapp.instagramUser"
                    hint="Seu usuário do Instagram (sem @)" />
                </div>
                <div v-if="whatsapp.type === 'instagram' && !isEdit"
                  class="text-center">
                  <q-btn flat
                    color="info"
                    class="bg-padrao"
                    icon="edit"
                    label="Nova senha"
                    @click="isEdit = !isEdit">
                    <q-tooltip>
                      Alterar senha
                    </q-tooltip>
                  </q-btn>
                </div>
                <div class="col-12"
                  v-if="whatsapp.type === 'instagram' && isEdit">
                  <c-input filled
                    label="Senha"
                    :type="isPwd ? 'password' : 'text'"
                    v-model="whatsapp.instagramKey"
                    hint="Senha utilizada para logar no Instagram"
                    placeholder="*************"
                    :disable="!isEdit">
                    <template v-slot:after>
                      <q-btn class="bg-padrao"
                        round
                        flat
                        color="negative"
                        icon="mdi-close"
                        @click="isEdit = !isEdit">
                        <q-tooltip>
                          Cancelar alteração de senha
                        </q-tooltip>

                      </q-btn>
                    </template>
                    <template v-slot:append>
                      <q-icon :name="isPwd ? 'visibility_off' : 'visibility'"
                        class="cursor-pointer"
                        @click="isPwd = !isPwd" />
                    </template>
                  </c-input>
                </div>
              </fieldset>

            </div>

          </div>
          <!-- <q-checkbox
            class="q-ml-md"
            v-model="whatsapp.isDefault"
            label="Padrão"
          /> -->
        </div>

        
        <div class="row q-my-md">
          <div>
            <div class="text-h8" v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') " style="margin-top: 20px;">ChatGPT</div>
            <div class="col-12 " v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') ">
              <label class="text-caption">ChatGPT API Key:</label>
              <input ref="apiKey"
                class="q-pa-sm bg-white full-width blur-effect"
                placeholder="Digite a API Key"
                dense
                outlined
                v-model="whatsapp.chatgptApiKey" />
            </div>
            <div class="col-12" v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') ">
              <label class="text-caption">ChatGPT Organization Key:</label>
              <input ref="orgKey"
                class="q-pa-sm bg-white full-width blur-effect"
                placeholder="Digite a Organization Key"
                dense
                outlined
                v-model="whatsapp.chatgptOrganizationId" />
            </div>
            <div class="col-12" v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') ">
              <label class="text-caption">Palavra para desligar o ChatGPT</label>
              <input ref="sairGpt"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite a Palavra para desligar o ChatGPT"
                dense
                outlined
                v-model="whatsapp.chatgptOff" />
            </div>
            <div class="col-12" v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') ">
              <label class="text-caption">Opção #1: Prompt ChatGPT:</label>
              <textarea ref="inputPrompt"
                style="min-height: 15vh; max-height: 15vh;"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite o Prompt"
                autogrow
                dense
                outlined
                v-model="whatsapp.chatgptPrompt" />
            </div>
            <label v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') " class="text-caption">Após mudar o Prompt, o histórico do ChatGPT deve ser restaurado clicando no botão abaixo:</label>
            <q-btn round
              flat
              dense
              @click="limparGpts()"
              v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') "
              >
              <q-icon size="2em"
                name="mdi-restore" />
              <q-tooltip>
                Restaurar Histórico ChatGPT
              </q-tooltip>
            </q-btn>
            <div class="col-12" v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') ">
              <label class="text-caption">Opção #2: Assistants ChatGPT:</label>
              <input ref="inputAssistant"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite a ID do Assistent"
                dense
                outlined
                v-model="whatsapp.assistantId" />
            </div>
            <label v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') " class="text-caption">Ao usar o Assistente o Prompt será desconsiderado</label>
            <q-btn round
              flat
              dense
              @click="assistenteNulo()"
              v-if="chatgptAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') "
              >
              <q-icon size="2em"
                name="mdi-restore" />
              <q-tooltip>
                Remover Assistente
              </q-tooltip>
            </q-btn>
          </div>
          <div>
            <div class="text-h8" v-if="typebotAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba') " style="margin-top: 20px;">TypeBOT</div>
            <div class="col-12" v-if="typebotAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba')">
              <label class="text-caption">Url do Typebot:</label>
              <input ref="inputAssistant"
                class="q-pa-sm bg-white full-width blur-effect"
                placeholder="Digite a Url do Typebot"
                dense
                outlined
                v-model="whatsapp.typebotUrl" />
            </div>
            <div class="col-12" v-if="typebotAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba')">
              <label class="text-caption">Nome do Bot no Typebot:</label>
              <input ref="inputAssistant"
                class="q-pa-sm bg-white full-width blur-effect"
                placeholder="Digite o Nome do Bot no Typebot"
                dense
                outlined
                v-model="whatsapp.typebotName" />
            </div>
            <div class="col-12" v-if="typebotAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba')">
              <label class="text-caption">Palavra para reiniciar o Typebot:</label>
              <input ref="inputAssistant"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite a Palavra para reiniciar o Typebot"
                dense
                outlined
                v-model="whatsapp.typebotRestart" />
            </div>
            <div class="col-12" v-if="typebotAtivo === 'enabled' && (whatsapp.type === 'whatsapp' || whatsapp.type === 'waba')">
              <label class="text-caption">Palavra para desligar o Typebot:</label>
              <input ref="inputAssistant"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite a Palavra para desligar o Typebot"
                dense
                outlined
                v-model="whatsapp.typebotOff" />     
            </div>
            <div class="col-12" v-if="typebotAtivo === 'enabled' && whatsapp.type === 'waba'">
              <q-item tag="label" v-ripple>
                <q-item-section>
                  <q-item-label>Habilitar botões</q-item-label>
                  <q-item-label caption> Habilitando esta opção serão enviados botões para as opções do Typebot, caso contrário, serão enviadas listas. </q-item-label>
                </q-item-section>

              <q-item-section avatar>
                <q-toggle
                    v-model="whatsapp.isButton"
                    false-value="disabled"
                    true-value="enabled"
                    checked-icon="check"
                    keep-color
                    :color="whatsapp.isButton === 'enabled' ? 'green' : 'negative'"
                    size="md"
                    unchecked-icon="clear"
                />
                </q-item-section>
              </q-item>
            </div>
          </div>
          <div>
            <div class="text-h8" v-if="dialogflowAtivo === 'enabled' && whatsapp.type === 'whatsapp' " style="margin-top: 20px;">DialogFlow</div>
            <div class="col-12" v-if="dialogflowAtivo === 'enabled' && whatsapp.type === 'whatsapp' ">
              <label class="text-caption">Project ID:</label>
              <input ref="projectId"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite o Project ID"
                dense
                outlined
                v-model="whatsapp.dialogflowProjectId" />
            </div>
            <div class="col-12" v-if="dialogflowAtivo === 'enabled' && whatsapp.type === 'whatsapp' ">
              <label class="text-caption">Idioma (Language):</label>
              <input ref="lang"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite o Idioma"
                dense
                outlined
                v-model="whatsapp.dialogflowLanguage" />
            </div>
            <div class="col-12" v-if="dialogflowAtivo === 'enabled' && whatsapp.type === 'whatsapp' ">
              <label class="text-caption">Palavra para desligar o DialogFlow</label>
              <input ref="sairDialog"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite a Palavra para desligar o DialogFlow"
                dense
                outlined
                v-model="whatsapp.dialogflowOff" />
            </div>
            <div class="col-12" v-if="dialogflowAtivo === 'enabled' && whatsapp.type === 'whatsapp' ">
              <label class="text-caption">Nome do Arquivo Json</label>
              <input ref="jsonNameDialog"
                class="q-pa-sm bg-white full-width"
                placeholder="Digite a Nome do Arquivo Json"
                dense
                outlined
                v-model="whatsapp.dialogflowJsonFilename" />
            </div>
            <div class="col-12" v-if="dialogflowAtivo === 'enabled' && whatsapp.type === 'whatsapp' ">
              <label class="text-caption">Conteúdo do Arquivo Json</label>
              <textarea ref="jsonDialog"
                style="min-height: 15vh; max-height: 15vh;"
                class="q-pa-sm bg-white full-width"
                placeholder="Copie o Conteúdo do Arquivo Json"
                autogrow
                dense
                outlined
                v-model="whatsapp.dialogflowJson" />
            </div>
          </div>

          <div class="text-h8" style="margin-top: 20px;">Despedida</div>
          <div class="col-12">
            <label class="text-caption">Mensagem de Despedida do Atendimento:</label>
            <textarea ref="inputFarewellMessage"
              style="min-height: 15vh; max-height: 15vh;"
              class="q-pa-sm bg-white full-width"
              placeholder="Digite a mensagem"
              autogrow
              dense
              outlined
              v-model="whatsapp.farewellMessage" />
          </div>
          <q-btn round
            v-if="whatsapp.type !== 'waba' "
            flat
            dense>
            <q-icon size="2em"
              name="mdi-variable" />
            <q-tooltip>
              Variáveis
            </q-tooltip>
            <q-menu touch-position>
              <q-list dense
                style="min-width: 100px">
                <q-item v-for="variavel in variaveis"
                  :key="variavel.label"
                  clickable
                  @click="onInsertSelectVariable(variavel.value)"
                  v-close-popup>
                  <q-item-section>{{ variavel.label }}</q-item-section>
                </q-item>
              </q-list>
            </q-menu>
          </q-btn>

          <div>
            <div class="col-12" v-if="whatsapp.type === 'whatsapp' || whatsapp.type === 'waba'">
              <q-item tag="label" v-ripple>
                <q-item-section>
                  <q-item-label>Habilitar avaliação automática</q-item-label>
                  <q-item-label caption> Habilitando esta opção será enviada a avaliação do atendimento de maneira automática na resolução de cada ticket. </q-item-label>
                </q-item-section>

              <q-item-section avatar>
                <q-toggle
                    v-model="whatsapp.sendEvaluation"
                    false-value="disabled"
                    true-value="enabled"
                    checked-icon="check"
                    keep-color
                    :color="whatsapp.sendEvaluation === 'enabled' ? 'green' : 'negative'"
                    size="md"
                    unchecked-icon="clear"
                />
                </q-item-section>
              </q-item>
            </div>

          </div>

        </div>
      </q-card-section>
      <q-card-actions align="center"
        class="q-mt-lg">
        <q-btn
          label="Sair"
          class="q-px-md q-mr-lg"
          color="negative"
          v-close-popup />
        <q-btn
          label="Salvar"
          color="primary"
          class="q-px-md"
          @click="handleSaveWhatsApp(whatsapp)" />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>
import { required, minLength, maxLength } from 'vuelidate/lib/validators'
import { UpdateWhatsapp, CriarWhatsapp, DeletarWhatsapp } from 'src/service/sessoesWhatsapp'
import cInput from 'src/components/cInput.vue'
import { copyToClipboard, Notify } from 'quasar'
import { ListarConfiguracoes } from 'src/service/configuracoes'
import { LimpartHistoricoGpt } from 'src/service/tickets'
import { VerificarTelefone } from 'src/service/waba'

export default {
  components: { cInput },
  name: 'ModalWhatsapp',
  props: {
    modalWhatsapp: {
      type: Boolean,
      default: false
    },
    whatsAppId: {
      type: Number,
      default: null
    },
    whatsAppEdit: {
      type: Object,
      default: () => { }
    }
  },
  data () {
    return {
      activeTab: 'tab1',
      showPairingCode: false,
      showProxy: false,
      isPwd: true,
      isEdit: false,
      chatgptAtivo: 'disabled',
      typebotAtivo: 'disabled',
      dialogflowAtivo: 'disabled',
      whatsapp: {
        name: '',
        wppUser: '',
        wppPass: '',
        proxyUrl: null,
        isDefault: false,
        tokenTelegram: '',
        instagramUser: '',
        instagramKey: '',
        tokenAPI: '',
        wabaId: '',
        bmToken: '',
        type: 'whatsapp',
        farewellMessage: '',
        wabaBSP: '360',
        chatgptPrompt: '',
        chatgptApiKey: '',
        chatgptOrganizationId: '',
        chatgptOff: '',
        assistantId: '',
        typebotRestart: '',
        isButton: true,
        typebotOff: '',
        typebotName: '',
        typebotUrl: '',
        dialogflowJsonFilename: '',
        dialogflowProjectId: '',
        dialogflowLanguage: '',
        dialogflowOff: '',
        dialogflowJson: '',
        wordlist: 'disabled',
        sendEvaluation: 'disabled'
      },
      optionsWhatsappsTypes: [
        { label: 'WhatsApp Oficial (WABA)', value: 'waba' },
        { label: 'WhatsApp Web (QRCode)', value: 'whatsapp' },
        { label: 'Telegram', value: 'telegram' },
        // { label: 'Instagram (Beta Version)', value: 'instagram' },
        // { label: 'Messenger (em breve)', value: 'messenger' }
      ],
      variaveis: [
        { label: 'Nome', value: '{{name}}' },
        { label: 'Saudação', value: '{{greeting}}' },
        { label: 'Protocolo', value: '{{protocol}}' },
        { label: 'E-mail (se existir)', value: '{{email}}' },
        { label: 'Telefone', value: '{{phoneNumber}}' },
        { label: 'Kanban', value: '{{kanban}}' },
        { label: 'Atendente', value: '{{user}}' },
        { label: 'E-mail Atendente', value: '{{userEmail}}' },
      ],
      FB: {},
      FBscope: {},
      FBLoginOptions: {
        scope:
          'pages_manage_metadata,pages_messaging,instagram_basic,pages_show_list,pages_read_engagement,instagram_manage_messages'
      },
      FBPageList: [],
      fbSelectedPage: { name: null, id: null },
      fbPageName: '',
      fbUserToken: ''
    }
  },
  validations: {
    whatsapp: {
      name: { required, minLength: minLength(3), maxLength: maxLength(50) },
      isDefault: {}
    }
  },
  computed: {
    cFbAppId () {
      return process.env.FACEBOOK_APP_ID
    },
    cBaseUrlIntegração () {
      return this.whatsapp.UrlMessengerWebHook
    }
  },
  methods: {
    handleSdkInit ({ FB }) {
      this.FB = FB
      // try login

      // this.FBscope = scope
    },
    async fbLogout (whatsapp) {
      console.info('fbLogout')
      await LogoutFacebookPages(whatsapp)
    },
    fbLogin (login, channel) {
      if (login?.status === 'connected') {
        this.fbFetchPages(
          login.authResponse.accessToken,
          login.authResponse.userID,
          channel
        )
        console.info('fbLogin in connected')
      } else if (login?.status === 'not_authorized') {
        // The person is logged into Facebook, but not your app.
        console.info('fbLogin in not_authorized')
      } else {
        // The person is not logged into Facebook, so we're not sure if
        // they are logged into this app or not.
        console.info('fbLogin in not logged')
      }
    },
    async fbFetchPages (_token, _accountId, channel) {
      try {
        const response = await FetchFacebookPages({
          whatsapp: channel,
          userToken: _token,
          accountId: _accountId
        })
        const {
          data: { data }
        } = response
        this.FBPageList = data.page_details
        this.fbUserToken = data.user_access_token
      } catch (error) {
        // Ignore error
      }
    },
    async assistenteNulo(){
      this.whatsapp.assistantId = null;
      const data = {
        ...this.whatsAppEdit,
        assistantId: null
      }
      await UpdateWhatsapp(this.whatsAppEdit.id, data)
    },
    async limparGpts(){
      const { data } = await LimpartHistoricoGpt()
    },
    async listarConfiguracoes() {
      const { data } = await ListarConfiguracoes()
      const chatgptConfig = data.find(config => config.key === 'chatgpt')
      this.chatgptAtivo = chatgptConfig.value
      const typebotConfig = data.find(config => config.key === 'typebot')
      this.typebotAtivo = typebotConfig.value
      const dialogflowConfig = data.find(config => config.key === 'dialogflow')
      this.dialogflowAtivo = dialogflowConfig.value
    },
    copy (text) {
      copyToClipboard(text)
        .then(this.$notificarSucesso('URL de integração copiada!'))
        .catch()
    },
    onInsertSelectVariable (variable) {
      const self = this
      var tArea = this.$refs.inputFarewellMessage
      // get cursor's position:
      var startPos = tArea.selectionStart,
        endPos = tArea.selectionEnd,
        cursorPos = startPos,
        tmpStr = tArea.value
      // filter:
      if (!variable) {
        return
      }
      // insert:
      self.txtContent = this.whatsapp.farewellMessage
      self.txtContent = tmpStr.substring(0, startPos) + variable + tmpStr.substring(endPos, tmpStr.length)
      this.whatsapp.farewellMessage = self.txtContent
      // move cursor:
      setTimeout(() => {
        tArea.selectionStart = tArea.selectionEnd = cursorPos + 1
      }, 10)
    },
    fecharModal () {
      this.whatsapp = {
        name: '',
        isDefault: false,
        wppUser: '',
        proxyUrl: null
      }
      this.showPairingCode = false
      this.showProxy = false
      this.$emit('update:whatsAppEdit', {})
      this.$emit('update:modalWhatsapp', false)
    },
    abrirModal () {
      if (this.whatsAppEdit.id) {
        this.whatsapp = { ...this.whatsAppEdit }
      }
    },
    async handleSaveWhatsApp (whatsapp) {
      this.$v.whatsapp.$touch()
      if (this.$v.whatsapp.$error) {
        return this.$q.notify({
          type: 'warning',
          progress: true,
          position: 'top',
          message: 'Ops! Verifique os erros...',
          actions: [{
            icon: 'close',
            round: true,
            color: 'white'
          }]
        })
      }
      try {
        if (this.whatsAppEdit.id) {
          await UpdateWhatsapp(this.whatsAppEdit.id, whatsapp)
          if(whatsapp.type === "waba"){
            try {
              const data = {
                tokenAPI: this.whatsapp.tokenAPI,
                wabaId: this.whatsapp.wabaId,
                bmToken: this.whatsapp.bmToken
              }
              const response = await VerificarTelefone(data);
            } catch (error) {
              if (error.response && error.response.status === 400) {
                this.$q.notify({
                  type: 'warning',
                  progress: true,
                  position: 'top',
                  message: 'Ops! Dados inválidos, verifique se os identificadores estão corretos...',
                  actions: [{ icon: 'close', round: true, color: 'white' }]
                });
              } else {
                this.$q.notify({
                  type: 'warning',
                  progress: true,
                  position: 'top',
                  message: 'Ops! Dados inválidos, verifique se os identificadores estão corretos...',
                  actions: [{ icon: 'close', round: true, color: 'white' }]
                });
                return
              }
            }
          }
        } else {
          await CriarWhatsapp(whatsapp)
          if(whatsapp.type === "waba"){
            try {
              const data = {
                tokenAPI: this.whatsapp.tokenAPI,
                wabaId: this.whatsapp.wabaId,
                bmToken: this.whatsapp.bmToken
              }
              const response = await VerificarTelefone(data);
            } catch (error) {
              if (error.response && error.response.status === 400) {
                this.$q.notify({
                  type: 'warning',
                  progress: true,
                  position: 'top',
                  message: 'Ops! Dados inválidos, verifique se os identificadores estão corretos...',
                  actions: [{ icon: 'close', round: true, color: 'white' }]
                });
              } else {
                this.$q.notify({
                  type: 'warning',
                  progress: true,
                  position: 'top',
                  message: 'Ops! Dados inválidos, verifique se os identificadores estão corretos...',
                  actions: [{ icon: 'close', round: true, color: 'white' }]
                });
                return
              }
            }
          }          
        }
        this.$q.notify({
          type: 'positive',
          progress: true,
          position: 'top',
          message: `Whatsapp ${this.whatsAppEdit.id ? 'editado' : 'criado'} com sucesso!`,
          actions: [{
            icon: 'close',
            round: true,
            color: 'white'
          }]
        })
        this.$emit('recarregar-lista')
        this.fecharModal()
      } catch (error) {
        console.error(error, error.data.error === 'ERR_NO_PERMISSION_CONNECTIONS_LIMIT')
        if (error.data.error === 'ERR_NO_PERMISSION_CONNECTIONS_LIMIT') {
          Notify.create({
            type: 'negative',
            message: 'Limite de conexões atingida.',
            caption: 'ERR_NO_PERMISSION_CONNECTIONS_LIMIT',
            position: 'top',
            progress: true
          })
        } else {
          console.error(error)
          return this.$q.notify({
            type: 'error',
            progress: true,
            position: 'top',
            message: 'Ops! Verifique os erros... O nome da conexão não pode existir na plataforma, é um identificador único.',
            actions: [{
              icon: 'close',
              round: true,
              color: 'white'
            }]
          })
        }
      }
    }
  },
  destroyed () {
    this.$v.whatsapp.$reset()
  },
  async mounted() {
    await this.listarConfiguracoes()
  }
}
</script>

<style lang="scss" scoped>
 .blur-effect {
    filter: blur(0px)   
  }
</style>
