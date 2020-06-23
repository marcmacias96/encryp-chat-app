<template>
 <div class="q-pa-md row justify-center">
    <div style="width: 100%; max-width: 400px">
      <q-chat-message
        v-for="message in messages"
        :key="message.id"
        :name="message.username"
        :text="['message.content']"
        :sent="message.username ===this.username ? true : false"
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
