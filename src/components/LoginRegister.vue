<template>
  <q-form @submit="submitForm">
    <q-input outlined v-model="username" label="Usuario" class="q-mb-md"/>
    <q-input outlined v-model="password" label="Contraseña" type="password" class="q-mb-md"/>
    <div class="row">
      <q-space/>
      <q-btn
      color="primary"
      type="submit"
      :label="tab"
      />
    </div>
  </q-form>
</template>

<script>
import gql from 'graphql-tag'
import { TokenService } from '../service/Storage'
export default {
  props: ['tab'],
  data () {
    return {
      username: '',
      password: ''
    }
  },
  methods: {
    submitForm () {
      if (this.tab === 'login') {
        try {
          this.$apollo.mutate({
            mutation: gql`
            mutation ($password: String!, $username: String!){
              signIn(password: $password, username: $username) {
                username
                key
              }
            }
            `,
            variables: {
              username: this.username,
              password: this.password
            }
          }).then(({ data }) => {
            const { username, key } = data.signIn
            TokenService.saveToken(JSON.stringify({
              username,
              key
            }))
            this.$router.push({
              path: '/chat',
              query: {
                username
              }
            })
          })
        } catch (error) {
          console.log(error)
        }
      } else {
        try {
          this.$apollo.mutate({
            mutation: gql`
            mutation ($password: String!, $username: String!){
              signUp(password: $password, username: $username) {
                username
              }
            }
            `,
            variables: {
              username: this.username,
              password: this.password
            }
          }).then(({ data }) => {
            console.log(data.data)
          })
        } catch (error) {
          console.log(error)
        }
      }
    }
  }
}
</script>

<style>

</style>
