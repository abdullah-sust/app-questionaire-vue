<template>
  <v-container class="pt-6">
    <v-row class="text-center">
      <v-col cols="12">
        <v-row justify="center">
          <v-alert
            v-if="isQuestionFound === false"
            dense
            outlined
            type="error"
          >
            No question found!
          </v-alert>
          <v-alert
            v-else
            color="primary"
            dark
            border="top"
            icon="mdi-head-question"
            transition="scale-transition"
          >
            {{ question }}
          </v-alert>
          <v-spacer></v-spacer>
          <template>
            <v-btn
              text
              v-if="isLoggedIn === true"
              color="primary"
              @click="clickedReply()"
            >
              Reply
            </v-btn>
          </template>
          <v-expansion-panels inset focusable v-if="isQuestionFound === true && answers.length > 0">
            <v-expansion-panel
              v-for="(answer, i) in answers" :key="i"
            >
              <v-expansion-panel-header>{{ answer.substring(0, 50) + ' . . .' }}</v-expansion-panel-header>
              <v-expansion-panel-content>
                {{ answer }}
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </v-row>
      </v-col>
    </v-row>
    <v-row class="text-center">
      <v-col class="col-12">
        <v-row>
          <v-alert
            v-if="isQuestionFound === true && answers.length === 0"
            dense
            outlined
            type="error"
          >
            No replies found!
          </v-alert>
        </v-row>
      </v-col>
    </v-row>
    <!-- DIALOG -->
    <v-dialog
      v-model="replyDialog"
      max-width="500px"
    >
      <v-card>
        <v-card-title>
          <span class="text-h5">Reply answer</span>
        </v-card-title>

        <v-card-text>
          <v-container>
            <v-form
              ref="form2"
              v-model="valid2"
              lazy-validation
            >
              <v-row>
                <v-col
                  cols="12"
                  sm="12"
                  md="12"
                >
                  <v-text-field
                    v-model="newAnswer"
                    :rules="answerRules"
                    counter="550"
                    label="Write your answer"
                    multiple-line
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-form>
          </v-container>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="blue darken-1"
            text
            @click="closeReply"
          >
            Cancel
          </v-btn>
          <v-btn
            color="blue darken-1"
            text
            :disabled="newAnswer === ''"
            @click="saveReply()"
          >
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!-- SNACKBAR -->
    <v-snackbar
      v-model="snackbar"
      color="primary"
      bottom
      right
    >
      {{ snackbarText }}

      <template v-slot:action="{ attrs }">
        <v-btn
          color="white"
          text
          v-bind="attrs"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'AnswerList',

  data: () => ({
    questionId: -1,
    snackbar: false,
    snackbarText: '',
    questionList: [{ id: 0, question: '', answer: [''] }],
    isQuestionFound: false,
    question: null,
    isLoggedIn: false,
    replyDialog: false,
    newAnswer: '',
    valid2: true,
    answers: [''],
    answerRules: [
      (value: any) => !!value || 'This field is required.',
      (value: any) => value.length <= 550 || 'Max 550 characters'
    ]
  }),

  created () {
    this.initialize()
  },

  methods: {
    initialize () {
      this.answers = []
      if (localStorage.getItem('isUserLoggedIn') === '1') {
        this.isLoggedIn = true
      } else {
        this.isLoggedIn = false
      }
      this.questionId = parseInt(this.$route.params.id)
      if (localStorage.getItem('question_list') !== null) {
        this.questionList = JSON.parse(localStorage.getItem('question_list') || '[]')
        this.questionList.forEach((item: any) => {
          if (item.id === this.questionId) {
            this.question = item.question
            this.answers = item.answer
            this.isQuestionFound = true
          }
        })
      }
    },

    clickedReply (id: any) {
      this.replyDialog = true
      this.questionId = id
    },

    closeReply () {
      this.newAnswer = ''
      this.replyDialog = false
    },

    saveReply () {
      this.answers.unshift(this.newAnswer)
      this.newAnswer = ''
      this.snackbarText = 'Successfully replied'
      this.snackbar = true
      this.replyDialog = false
      localStorage.setItem('question_list', JSON.stringify(this.questionList))
    }
  }
})
</script>
