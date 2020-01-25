/* eslint-disable no-console */ /* eslint-disable no-debugger */
<template>
  <div>
    <v-snackbar v-model="snackbar" :color="color" :timeout="timeout" top="top">
      {{ text }}
      <v-btn @click="snackbar = false" dark text>
        Close
      </v-btn>
    </v-snackbar>
    <blockquote class="blockquote">
      <v-flex>
        <h2>Welcome {{ username }}</h2>
      </v-flex>
      <v-row justify="center">
        <v-card-text>Please pick the numbers from below</v-card-text>
        <v-col
          v-for="n in allowedCards"
          :key="n"
          @click="addCard(n)"
          cols="auto"
          class="card-col"
        >
          <v-card height="50" width="50">
            <v-row
              v-text="n"
              class="fill-height"
              align="center"
              justify="center"
            >
            </v-row>
          </v-card>
        </v-col>
      </v-row>
      <v-row justify="center">
        <v-card-text
          >You have selected {{ selectedCards.length }} cards:</v-card-text
        >
        <v-col
          v-for="(n, index) in selectedCards"
          :key="index"
          cols="auto"
          class="card-col"
        >
          <v-card height="50" width="50">
            <v-row
              v-text="n"
              class="fill-height"
              align="center"
              justify="center"
            >
            </v-row>
            <v-icon
              @click="removeCard(index)"
              class="close-btn"
              right
              style="font-size:10px"
              color="error"
              >mdi-close</v-icon
            >
          </v-card>
        </v-col>
      </v-row>
      <v-row justify="center">
        <v-btn
          @click="generateCards"
          v-if="playActive"
          outlined
          color="primary"
          text
          >Play</v-btn
        >
        <v-btn @click="resetForm" v-else outlined color="primary" text
          >Play Again</v-btn
        >
      </v-row>
      <v-row justify="center">
        <h2>{{ playerOnePoints }} : {{ playerTwoPoints }}</h2>
      </v-row>
      <!-- <template>
        <v-form ref="form" v-model="valid">
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="cardNo"
                  :rules="cardNoRules"
                  label="Enter card numbers, pick from the allowed"
                  required
                ></v-text-field>
                <v-slide-x-reverse-transition>
                  <v-tooltip left>
                    <template v-slot:activator="{ on }">
                      <v-btn @click="resetForm" v-on="on" icon class="my-0">
                        <v-icon>mdi-refresh</v-icon>
                      </v-btn>
                    </template>
                    <span>Refresh form</span>
                  </v-tooltip>
                </v-slide-x-reverse-transition>
                <v-btn @click="generateCards" outlined color="primary" text
                  >Play</v-btn
                >
              </v-col>
            </v-row>
          </v-container>
        </v-form>
      </template> -->
    </blockquote>
    <v-row v-if="generatedCards.length > 0" justify="center">
      <v-card-text
        >The generated {{ generatedCards.length }} cards are</v-card-text
      >
      <v-col
        v-for="(n, index) in generatedCards"
        :key="index"
        cols="auto"
        class="card-col"
      >
        <v-card height="50" width="50">
          <v-row v-text="n" class="fill-height" align="center" justify="center">
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'UserGame',
  props: { username: { type: String, required: true } },
  data: () => ({
    valid: false,
    cardNo: '',
    playerOnePoints: 0,
    playerTwoPoints: 0,
    text: '',
    timeout: 3000,
    color: '',
    snackbar: false,
    playActive: true,
    gameData: {
      username: null,
      users_score: null,
      opp_score: null,
      won: null
    },
    cardNoRules: [
      (v) => !!v || 'Cards is required',
      (v) =>
        (v && v.length <= 1) ||
        'Cards numbers must be greater than 1 characters'
    ],
    allowedCards: [
      '2',
      '3',
      '4',
      '5',
      '6',
      '7',
      '8',
      '9',
      '10',
      'J',
      'Q',
      'K',
      'A'
    ],
    selectedCards: [],
    generatedCards: []
  }),
  //
  computed: {},

  watch: {},

  methods: {
    addCard(num) {
      this.selectedCards.push(num)
    },
    removeCard(index) {
      this.selectedCards.splice(index, 1)
    },
    resetForm() {
      // this.$refs.form.reset()
      this.selectedCards = []
      this.generatedCards = []
      this.playerOnePoints = 0
      this.playerTwoPoints = 0
      this.playActive = true
    },
    submit() {
      if (this.$refs.form.validate()) {
        this.$emit('generateCards', this.cardNo)
      }
    },
    async generateCards(myArray) {
      const t = JSON.parse(JSON.stringify(this.allowedCards))
      this.generatedCards = await this.handsCards(t)
      await this.calculatePoints()
    },
    getRandomInt(n) {
      return Math.floor(Math.random() * n)
    },
    handsCards(arr) {
      const length = this.selectedCards.length
      const n = arr.length // Length of the array
      const hands = []

      for (let i = 0; i < n - 1; ++i) {
        const j = this.getRandomInt(n) // Get random of [0, n-1]

        const temp = arr[i] // Swap arr[i] and arr[j]
        arr[i] = arr[j]
        arr[j] = temp
      }

      for (let i = 0; i < length; i++) {
        const j = this.getRandomInt(n)
        hands.push(arr[j])
      }

      return hands // Return shuffled string
    },
    calculatePoints() {
      const all = this.allowedCards
      const sel = this.selectedCards
      const gen = this.generatedCards
      for (let i = 0; i < sel.length; i++) {
        const indSel = all.indexOf(sel[i])
        const indGen = all.indexOf(gen[i])
        if (indSel > indGen) this.playerOnePoints++
        else this.playerTwoPoints++
      }
      this.setGameForm(this.playerOnePoints, this.playerTwoPoints)
    },
    async setGameForm(pointOne, pointTwo) {
      this.gameData.username = this.username
      this.gameData.users_score = pointOne
      this.gameData.opp_score = pointTwo
      if (pointOne > pointTwo) {
        this.gameData.won = 1
        this.snackbar = true
        this.color = 'cyan darken-2'
        this.text = 'Hurray, You have won the game'
      } else {
        this.gameData.won = 0
        this.snackbar = true
        this.color = 'error'
        this.text = 'Hurray, You have lost the game'
      }

      await this.insertRecord()
    },
    insertRecord() {
      return axios
        .post(`https://cards.dev/api/v1/games`, this.gameData)
        .then((res) => {
          if (res.data.response === 'success') {
            this.playActive = false
          }
        })
        .catch((e) => {
          // eslint-disable-next-line no-console
          console.log(e)
        })
    }
  }
  //
}
</script>
<style>
.card-col {
  cursor: pointer;
}
.close-btn {
  position: absolute;
  top: 0;
  right: 0%;
  width: 2x;
  height: 2x;
}
</style>
