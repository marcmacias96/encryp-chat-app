<template>
<div class="full-width row  justify-center" style="height: 750px;">
  <div class="column col-3">
    <online-users />
  </div>
  <div class="flex column col justify-end">
  <list-message :username="username" :messages="messages"/>
  </div>
  <q-footer elevated>
    <q-toolbar>
      <q-form @submit="sendMessage" class="full-width">
        <q-input
          v-model="newMessage"
          bg-color="white"
          rounded
          outlined
          label="Mensaje"
          dense
          >
          <template v-slot:after>
            <q-btn round dense flat icon="send" color="white" />
          </template>
        </q-input>
      </q-form>
    </q-toolbar>
  </q-footer>
</div>

</template>

<script>
import gql from 'graphql-tag'
import { TokenService } from '../service/Storage'
import Encrypt from 'crypto-js'
import NodeRSA from 'node-rsa'

export default {
  name: 'PageIndex',
  components: {
    'online-users': require('components/OnlineUsers.vue').default,
    'list-message': require('components/ListMessage.vue').default
  },
  props: {
    username: {
      type: String,
      default () {
        const savedData = JSON.parse(TokenService.getToken())
        return savedData.username
      }
    },
    secret: {
      type: String,
      default () {
        const savedData = JSON.parse(TokenService.getToken())
        return savedData.secret
      }
    },
    Pkey: {
      type: String,
      default () {
        const savedData = JSON.parse(TokenService.getToken())
        return savedData.key
      }
    }
  },
  created () {
    setInterval(
      () => {
        this.$apollo.mutate({
          mutation: gql`
          mutation ($username: String!, $time: timestamp!) {
            update_user(_set: {last_seen: $time}, where: {username: {_eq: $username}}) {
              affected_rows
            }
          }
          `,
          variables: {
            username: this.username,
            time: 'now()'
          }
        })
      },
      3000
    )
  },
  data () {
    return {
      newMessage: '',
      messages: [],
      ecryptmsg: ''
    }
  },
  methods: {
    sendMessage () {
      if (this.newMessage === '') return
      const Pkey = NodeRSA(this.Pkey)
      this.ecryptmsg = Encrypt.AES.encrypt(this.newMessage, Pkey.decrypt(this.secret, 'utf8')).toString()
      this.$apollo.mutate({
        mutation: gql`
        mutation ($message: String, $username: String, $timestamp: timestamp) {
          insert_message(objects: {username: $username, content: $message, timestamp: $timestamp}) {
            affected_rows
          }
        }
        `,
        variables: {
          username: this.username,
          message: this.ecryptmsg,
          timestamp: 'now()'
        }
      }).then(({ data }) => {
      })
      this.newMessage = ''
      this.ecryptmsg = ''
    }
  },
  apollo: {
    messages: {
      query: gql`
      query ($time: timestamp) {
        message(order_by: {timestamp: asc}, where: {_and: {id: {_neq: -1}, timestamp: {_gte: $time}}}, limit: 8) {
          id
          content
          username
        }
      }
      `,
      variables () {
        return {
          time: '2020-06-22T16:57:25.918155'
        }
      },
      update (data) {
        const Pkey = NodeRSA(this.Pkey)
        const recivedMes = data.message.map((item) => {
          if (item.content.length === 44) {
            const bytes = Encrypt.AES.decrypt(item.content, Pkey.decrypt(this.secret, 'utf8'))
            item.content = bytes.toString(Encrypt.enc.Utf8)
          }
          return item
        })
        return recivedMes
      },
      subscribeToMore: {
        document: gql`
        subscription {
          message ( order_by: {id:desc} limit: 1) {
            id
            username
            content
            timestamp
          } 
        }
        `,
        updateQuery: (previousResult, { subscriptionData }) => {
          const { TokenService } = require('../service/Storage')
          const savedData = JSON.parse(TokenService.getToken())
          const Pkey = NodeRSA(savedData.key)
          if (previousResult && !previousResult.message.some(item => item.id === subscriptionData.data.message[0].id)) {
            if (subscriptionData.data.message.length > 0) {
              const bytes = Encrypt.AES.decrypt(subscriptionData.data.message[0].content, Pkey.decrypt(savedData.secret, 'utf8'))
              subscriptionData.data.message[0].content = bytes.toString(Encrypt.enc.Utf8)
            }
            return {
              message: [
                ...previousResult.message,
                ...subscriptionData.data.message
              ]
            }
          }
        }
      },
      error () {
        alert('Error occured')
      }
    }
  }
}
</script>
