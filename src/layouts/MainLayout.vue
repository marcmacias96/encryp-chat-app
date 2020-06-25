<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>

        <q-toolbar-title class="absolute-center">
          Chat Encriptado
        </q-toolbar-title>

        <q-btn
          class="absolute-right q-pr-sm"
          icon="account_circle"
          no-caps
          flat
          dense
          :label="title"
          @click="logOut"
        />

      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { TokenService } from '../service/Storage'
export default {
  name: 'MainLayout',
  computed: {
    title () {
      const currentPath = this.$router.fullPath
      if (currentPath === '/') return ''
      return 'Salir'
    }
  },
  data () {
    return {
      leftDrawerOpen: false
    }
  },
  methods: {
    logOut () {
      TokenService.removeToken()
      this.$router.push({
        path: '/'
      })
    }
  }
}
</script>
