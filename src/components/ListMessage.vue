<template>
 <div class="q-pa-md row justify-center">
    <div style="width: 100%;">
      <q-chat-message
        v-for="message in messages"
        :key="message.id"
        :name="message.username"
        :text="[message.content]"
        :sent="message.username ===username ? true : false"
      />
    </div>
 </div>
</template>

<script>
export default {
  props: {
    username: {
      type: String
    },
    messages: {
      type: Array
    }
  }

}
</script>

<style>

</style>
