<template>
    <div class="q-pa-md" style="max-width: 350px">
    <q-toolbar class="bg-primary text-white shadow-2">
      <q-toolbar-title>Usuarios</q-toolbar-title>
    </q-toolbar>

    <q-list bordered>
      <q-item v-for="user in onlineUsers" :key="user.id" class="q-my-sm" clickable v-ripple>
        <q-item-section avatar>
          <q-avatar color="primary" text-color="white">
            {{ user.username.charAt(0) }}
          </q-avatar>
        </q-item-section>

        <q-item-section>
          <q-item-label>{{ user.username }}</q-item-label>
        </q-item-section>
      </q-item>
    </q-list>
  </div>
</template>

<script>
import gql from 'graphql-tag'
export default {
  data () {
    return {
      onlineUsers: []
    }
  },
  apollo: {
    $subscribe: {
      users: {
        query: gql`
        subscription {
         online_users(order_by: {username: asc}) {
            username
            id
         }
        }`,
        result ({ data, loading, error }) {
          if (error) {
            console.log(error)
          }
          this.onlineUsers = data.online_users
        }
      }
    }
  }
}
</script>

<style>

</style>
