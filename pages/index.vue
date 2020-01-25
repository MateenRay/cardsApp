<template>
  <v-layout>
    <v-flex class="text-center">
      <UserName v-if="!isGame" @onSubmit="submit" />
      <UserGame v-if="isGame" :username="username" />
      <v-data-table
        :headers="headers"
        :items="games"
        :sort-by="['created_at']"
        :sort-desc="[true]"
        class="elevation-1"
      ></v-data-table>
    </v-flex>
  </v-layout>
</template>
<script>
import axios from 'axios'
import UserName from '~/components/UserName/index.vue'
import UserGame from '~/components/UserGame/index.vue'
export default {
  components: {
    UserName,
    UserGame
  },
  data: () => ({
    isGame: false,
    games: [],
    headers: [
      {
        text: 'Player Name',
        align: 'left',
        sortable: true,
        value: 'username'
      },
      { text: 'Player Score', value: 'users_score', align: 'center' },
      { text: 'Hands Score', value: 'opp_score', align: 'center' },
      { text: 'Won', value: 'won' },
      {
        text: 'Played On',
        value: 'created_at',
        sortable: true,
        align: 'center'
      }
    ]
  }),
  asyncData({ params, error }) {
    return axios
      .get(`https://cards.dev/api/v1/games/`)
      .then((res) => {
        return { games: res.data.data }
      })
      .catch((e) => {
        // eslint-disable-next-line no-console
        console.log(e)
      })
  },
  methods: {
    submit(value) {
      this.isGame = true
      this.username = value
    }
  }
}
</script>
