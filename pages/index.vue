<template>
  <v-layout
    column
    justify-center
    align-center
  >
    <v-flex xs12 sm8>
      <v-card min-width="400">
        <v-snackbar
          v-model="snackbar"
          :timeout="6000"
          top>
          {{ message }}
          <v-btn
            color="blue"
            text
            @click="snackbar = false"
          >
            Close
          </v-btn>
        </v-snackbar>
        <v-card-title>
          <h1>Nuxt chat</h1>
        </v-card-title>
        <v-card-text>
          <v-form
            ref="form"
            v-model="valid"
            lazy-validation
          >
            <v-text-field
              v-model="name"
              :counter="16"
              :rules="nameRules"
              label="Ваше имя"
              required
            />

            <v-text-field
              v-model="room"
              :rules="roomRules"
              label="Введите комнату"
              required
            />

            <v-btn
              :disabled="!valid"
              color="primary"
              class="mr-4"
              @click="submit"
            >
              Войти
            </v-btn>
          </v-form>
        </v-card-text>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapMutations } from 'vuex'
export default {
  layout: 'empty',
  head: () => ({
    title: 'Welcom to NUXT chat'
  }),
  data: () => ({
    valid: true,
    name: '',
    snackbar: false,
    message: '',
    nameRules: [
      v => !!v || 'Name is required',
      v => (v && v.length <= 16) || 'Name must be less than 10 characters'
    ],
    room: '',
    roomRules: [
      v => !!v || 'Введите комнату'
    ]
  }),
  sockets: {
    connect () {
      console.log('socket connected')
    }
  },
  mounted () {
    const { message } = this.$route.query
    if (message === 'noUser') {
      this.message = 'Enter params'
    } else if (message === 'leftChat') {
      this.message = 'You are leave chat'
    }

    this.snackbar = !!this.message
  },
  methods: {
    ...mapMutations(['setUser']),
    submit () {
      if (this.$refs.form.validate()) {
        const user = {
          name: this.name,
          room: this.room
        }

        this.$socket.emit('userJoined', user, (data) => {
          if (typeof data === 'string') {
            console.error(data)
          } else {
            user.id = data.userId
            this.setUser(user)
            this.$router.push('/chat')
          }
        })
      }
    }
  }
}
</script>
