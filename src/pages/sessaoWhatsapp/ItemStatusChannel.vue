<template>
  <div>
    <q-item>
      <q-item-section avatar>
        <q-icon v-if="item.status == 'qrcode'"
          color="primary"
          name="mdi-crop-free"
          size="2.5em" />
        <q-icon v-if="item.status == 'DISCONNECTED'"
          color="negative"
          size="2.5em"
          name="mdi-wifi-alert" />
        <q-icon name="mdi-wifi-arrow-up-down"
          color="positive"
          size="2.5em"
          v-if="item.status == 'CONNECTED'" />
        <q-icon v-if="['PAIRING', 'TIMEOUT'].includes(item.status)"
          color="negative"
          size="2.5em"
          name="mdi-wifi-strength-1-alert" />
        <q-spinner v-if="item.status == 'OPENING'"
          color="green-7"
          size="3em"
          :thickness="2" />
      </q-item-section>
      <q-item-section>
        <q-item-label v-if="item.status == 'qrcode'">
          <span class="text-weight-medium"> Esperando leitura do QR Code </span>
          <span class="row col"> Clique no botão 'QR CODE' e leia o QR Code com o seu celular para iniciar a sessão
          </span>
        </q-item-label>
        <q-item-label v-if="item.status == 'DISCONNECTED'">
          <span class="text-weight-medium"> Falha ao iniciar comunicação para este canal. </span>
          <span class="row col"
            v-if="item.type === 'whatsapp'"> Certifique-se de que seu celular esteja conectado à internet e tente
            novamente, ou solicite um novo QR Code </span>
          <span class="row col"
            v-if="item.type === 'waba'"> Tente conectar novamente. Caso o erro permaneça, confirme se os tokens estão
            corretos. </span>
          <span class="row col"
            v-if="item.type === 'telegram'"> Tente conectar novamente. Caso o erro permaneça, confirme se o token está
            correto.</span>
          <span class="row col"
            v-if="item.type === 'instagram'"> Tente conectar novamente. Caso o erro permaneça, confirme se as
            credenciais estão corretas.</span>
        </q-item-label>
        <q-item-label v-if="item.status == 'CONNECTED' && item.type === 'waba'">  
          <span class="text-weight-medium blur-effect"> Conexão estabelecida: {{item.tokenAPI}}</span>
        </q-item-label>
        <q-item-label v-if="item.status == 'CONNECTED' && item.type === 'whatsapp'">  
          <span class="text-weight-medium blur-effect">
            Conexão estabelecida: 
            <template v-if="item.number" >{{ item.number }}</template>
            <template v-else>Carregando dados...</template>
          </span>
        </q-item-label>
        <q-item-label v-if="item.status == 'CONNECTED' && item.type === 'whatsapp'">
          <template v-if="item.profilePic">
            <img :src="item.profilePic" alt="Perfil" style="width: 30px; height: 30px; object-fit: cover; border-radius: 50%; vertical-align: middle; margin-right: 5px;">
          </template>
          <template v-else>
            <span class="mdi mdi-account" style="font-size: 30px; height: 30px; width: 30px; border-radius: 50%; vertical-align: middle; margin-right: 5px;"></span>
          </template>
          <span class="text-weight-medium">
            <template v-if="item.phone && item.phone.pushname">{{ item.phone.pushname }}</template>
            <template v-else>Carregando dados...</template>
          </span>
        </q-item-label>
        <!-- <q-item-label v-if="item.status == 'CONNECTED' && item.type === 'whatsapp'">  
          <span class="text-weight-medium"> Conexão estabelecida: {{item.number}}</span>
        </q-item-label>
        <q-item-label v-if="item.status == 'CONNECTED' && item.type === 'whatsapp'">
          <img :src="item.profilePic" alt="Perfil" style="width: 30px; height: 30px; object-fit: cover; border-radius: 50%; vertical-align: middle; margin-right: 5px;">
          <span class="text-weight-medium"> {{item.phone.pushname}}</span>
        </q-item-label> -->
        <q-item-label v-if="['PAIRING', 'TIMEOUT'].includes(item.status)">
          <span class="text-weight-medium"> A conexão com o celular foi perdida </span>
          <span class="row col"> Certifique-se de que seu celular esteja conectado à internet e o WhatsApp esteja
            aberto, ou clique no botão 'Desconectar' para obter um novo QR Code </span>
        </q-item-label>
        <q-item-label v-if="item.status == 'OPENING'">
          <span class="text-weight-medium"> Estabelecendo conexão. </span>
          <span class="row col"> Isso poderá demorar um pouco... </span>
        </q-item-label>
        <q-item-label caption>
          Última Atualização: {{ formatarData(item.updatedAt, 'dd/MM/yyyy HH:mm') }}
        </q-item-label>
      </q-item-section>
    </q-item>
  </div>
</template>
<script>
import { format, parseISO } from 'date-fns'
import pt from 'date-fns/locale/pt-BR/index'

export default {
  name: 'ItemStatusChannel',
  props: {
    item: {
      type: Object,
      default: () => { }
    }
  },
  methods: {
    formatarData (data, formato) {
      return format(parseISO(data), formato, { locale: pt })
    }
  }
}
</script>

<style lang="sass">
.blur-effect 
  filter: blur(0px)    
</style>