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
                secret
              }
            }
            `,
            variables: {
              username: this.username,
              password: this.password
            }
          }).then(({ data }) => {
            const { username, key, secret } = data.signIn
            if (username === 'no existe') {
              alert('Usiario no existe')
            } else {
              TokenService.saveToken(JSON.stringify({
                username,
                key,
                secret
              }))
              this.$router.push({
                path: '/chat'
              })
            }
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
            alert('Usuario Registrado')
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
