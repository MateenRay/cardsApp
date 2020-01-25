<template>
  <div>
    <blockquote class="blockquote">
      <v-form ref="form" v-model="valid">
        <v-container>
          <v-row>
            <v-col cols="12">
              <v-text-field
                v-model="name"
                :rules="nameRules"
                label="Enter your full name"
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
              <v-btn @click="submit" color="primary" text>Continue</v-btn>
            </v-col>
          </v-row>
        </v-container>
      </v-form>
    </blockquote>
  </div>
</template>

<script>
export default {
  name: 'UserName',
  data: () => ({
    valid: false,
    name: '',
    nameRules: [
      (v) => !!v || 'Name is required',
      (v) => (v && v.length >= 3) || 'Name must be greater than 3 characters'
    ]
  }),
  //
  computed: {},

  watch: {},

  methods: {
    resetForm() {
      this.$refs.form.reset()
    },
    submit() {
      if (this.$refs.form.validate()) {
        this.$emit('onSubmit', this.name)
      }
    }
  }
  //
}
</script>

<style></style>
