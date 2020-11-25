<template>
  <div style="max-width: 800px">
    <amplify-authenticator>
      <v-text-field
        v-model="form.comment"
        label="コメント"
        placeholder="ここにコメントを書きましょう"
        outlined
        class="mx-auto"
        append-icon="mdi-check-bold"
        style="max-width: 100%; box-sizing: border-box"
        @keydown="onEnter"
        @click:append="createChat"
      ></v-text-field>
    </amplify-authenticator>

    <v-card v-for="(item, index) in items" :key="index" tile>
      <v-list-item two-line>
        <v-list-item-content>
          <v-list-item-title>{{ item.comment }}</v-list-item-title>
          <v-list-item-subtitle>by: {{ item.owner }}</v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
    </v-card>

    <v-card>
      <amplify-sign-out v-if="logoutBtn"></amplify-sign-out>
    </v-card>
  </div>
</template>
<script>
import { API } from 'aws-amplify'
import { onAuthUIStateChange } from '@aws-amplify/ui-components'
import { createChat } from '~/graphql/mutations'
import { listChats } from '~/graphql/queries'
import { onCreateChat } from '~/graphql/subscriptions'
export default {
  data() {
    return {
      form: {
        comment: '',
      },
      items: [],
      logoutBtn: false,
    }
  },
  created() {
    this.getChatList()
    this.subscribe()

    onAuthUIStateChange((authState, authData) => {
      if (authState === 'signedin') {
        this.getChatList()
        this.subscribe()
        this.logoutBtn = true
      } else {
        this.items = []
        this.logoutBtn = false
      }
    })
  },
  methods: {
    async createChat() {
      const comment = this.form.comment
      if (!comment) return
      const chat = { comment }

      await API.graphql({
        query: createChat,
        variables: { input: chat },
      })
      this.form.comment = ''
    },
    onEnter(event) {
      if (event.keyCode !== 13) return
      this.createChat()
    },
    async getChatList() {
      const chatList = await API.graphql({
        query: listChats,
      })
      this.items = chatList.data.listChats.items
    },
    subscribe() {
      API.graphql({ query: onCreateChat }).subscribe({
        next: (eventData) => {
          const chat = eventData.value.data.onCreateChat
          if (this.items.some((item) => item.comment === chat.comment)) return
          this.items = [...this.items, chat]
        },
      })
    },
  },
}
</script>
