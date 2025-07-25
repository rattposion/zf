<template>
  <div>
    <q-tabs class="text-teal" align="left">
      <q-route-tab
        to="/massa/texto"
        name="bulktext"
        no-caps
        icon="mdi-rocket"
        label="Disparo em Massa"
      />
      <q-route-tab
        to="/massa/textovariavel"
        name="bulktextVariavel"
        no-caps
        icon="mdi-rocket"
        label="Disparo em Massa + Variavel"
      />
      <q-route-tab
        to="/massa/sms"
        name="bulktextSMS"
        no-caps
        icon="mdi-rocket"
        label="Disparo em Massa SMS"
      />
    </q-tabs>
    <router-view />
  </div>
</template>

<script>
const usuario = JSON.parse(localStorage.getItem('usuario'))
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'IndexConfiguracoes',
  data() {
    return {
      userProfile: 'user',
      usuario
    }
  },
  methods: {

  },
  async mounted() {
    this.userProfile = localStorage.getItem('profile')
  },
})
</script>
