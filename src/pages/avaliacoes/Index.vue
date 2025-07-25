<template>
  <div v-if="userProfile === 'admin' || userProfile === 'super' ">
    <q-tabs class="text-teal" align="left">
      <q-route-tab
        to="/avaliacoes/listar"
        name="list"
        no-caps
        icon="mdi-clipboard-list-outline"
        label="Listar"
      />
      <q-route-tab
        to="/avaliacoes/configurar"
        name="config"
        no-caps
        icon="mdi-cog"
        label="Configurar"
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
