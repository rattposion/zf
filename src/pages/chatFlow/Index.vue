<template>
  <div v-if="userProfile === 'admin' || userProfile === 'super' ">
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'IndexChatFlow',
  data () {
    return {
      userProfile: 'user',
    }
  },
  mounted () {
    this.userProfile = localStorage.getItem('profile')
  }
}
</script>

<style lang="scss" scoped>
</style>
