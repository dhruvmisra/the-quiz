<template>
  <transition name="fade">
    <section class="outer" v-if="!hasEnded">
      <div class="home" v-if="!secondplayer">
        <div class="container">
          <div class="logo">            
            <h1 style="font-size: 10em">Q</h1>
            <h4 class="display-4">The Quiz</h4>
          </div>

          <div class="row justify-content-around pt-5 my-5">
            <div class="idDiv mb-5">
              <p>Share this ID with your friend to connect</p>
              <h1 class="text center">{{ idToShow }}</h1>
            </div>

            <div class="inputDiv">
              <p>Or enter your friend's ID to start playing</p>
              <input
                class="form-control py-4 px-3 mx-auto"
                type="number"
                placeholder="Enter ID to connect"
                v-model="idToConnect"
              />
              <button
                class="btn btn-danger rounded-0 px-5 py-2 mt-3 w-100"
                type="button"
                :disabled="idToConnect == null"
                @click="connectToChannel(idToConnect)"
              >Connect</button>
            </div>
          </div>
        </div>
      </div>

      <div class="play" v-else-if="secondplayer">
        <div class="users-head">
          <p class="mx-auto">The Quiz</p>

          <div class="container row mx-auto">
            <div class="mx-auto">
              <div class="row">
                <div class="user-img rounded-circle border"></div>
                <h5 class="my-auto mx-3">User 1</h5>
              </div>
              <p class="mt-1 mb-3">Score: {{playerdata.one.score}}</p>
            </div>

            <div v-if="secondplayer" class="mx-auto">
              <div class="row">
                <div class="user-img rounded-circle border"></div>
                <h5 class="my-auto mx-3">User 2</h5>
              </div>
              <p class="mt-1 mb-3">Score: {{playerdata.two.score}}</p>
            </div>
          </div>

          <p class="text-center pb-1">Time: {{ timeLeft }}s</p>
          <div class="progress w-100">
            <div
              class="progress-bar bg-warning"
              role="progressbar"
              :style="{width: this.timeLeft/10*100 + '%'}"
              aria-valuenow="75"
              aria-valuemin="0"
              aria-valuemax="100"
            ></div>
          </div>
        </div>

        <div class="container mt-4 pb-5">
          <p class="question" :class="{'question-after': hasAnswered }">{{question.ques}}</p>
          <div class="container mt-3">
            <transition name="fade">
              <div class="options row justify-content-center mx-auto" v-if="show == true">
                <button
                  class="btn btn-option m-2 w-25 p-5"
                  :class="{ 'wronganswer': hasAnswered && !item.correct, 'correctanswer': hasAnswered && item.correct}"
                  @click="checkAnswer(item)"
                  v-for="item in options"
                  :key="item.text"
                >{{item.text}}</button>
              </div>
            </transition>
          </div>
        </div>
      </div>
    </section>

    <section v-else>
      <div class="container p-5 text-dark">
        <div class="card p-5">
          <div class="card-body">
            <h1 class="text-center text-success my-4 pb-5">User {{ winner.userid }} wins!</h1>
            <div class="winners row justify-content-center">
              <div class="p-5" :class="{ 'winner': winner.userid == 1 }">
                <div class="user-img rounded-circle border"></div>
                <h5 class="my-auto mx-3">User 1</h5>
                <p class="mt-1 mb-3">Final Score: {{playerdata.one.score}}</p>
              </div>

              <h2 class="mx-4 my-auto">VS</h2>

              <div class="p-5" :class="{ 'winner': winner.userid == 2 }">
                <div class="user-img rounded-circle border"></div>
                <h5 class="my-auto mx-3">User 2</h5>
                <p class="mt-1 mb-3">Final Score: {{playerdata.two.score}}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </transition>
</template>

<script>
import ChannelDetails from "@/components/ChannelDetails";

const questions = [
  {
    ques:
      "Which is the World's Largest desert?",
    options: [
      { text: "Thar", correct: false },
      { text: "Kalaharin", correct: false },
      { text: "Sahara", correct: true },
      { text: "Sonoran", correct: false }
    ],
    answer: "Sahara"
  },
  {
    ques:
      "Country that has the highest in Barley Production?",
    options: [
      { text: "China", correct: false },
      { text: "India", correct: false },
      { text: "France", correct: false },
      { text: "Russia", correct: true }
    ],
    answer: "Russia"
  },
  {
    ques:
      "The metal whose salts are sensitive to light is?",
    options: [
      { text: "Zinc", correct: false },
      { text: "Silver", correct: true },
      { text: "Copper", correct: false },
      { text: "Aluminium", correct: false }
    ],
    answer: "Silver"
  },
  {
    ques:
      "Mount Everest is located in?",
    options: [
      { text: "Nepal", correct: true },
      { text: "India", correct: false },
      { text: "Tibet", correct: false },
      { text: "China", correct: false }
    ],
    answer: "Nepal"
  },
  {
    ques:
      "Which soil is suitable for agriculture?",
    options: [
      { text: "Red soil", correct: false },
      { text: "Sand", correct: false },
      { text: "Black soil", correct: false },
      { text: "Peaty soil", correct: true }
    ],
    answer: "Peaty soil"
  },
  {
    ques:
      "The device used for measuring altitudes is?",
    options: [
      { text: "altimeter", correct: true },
      { text: "ammeter", correct: false },
      { text: "audiometer", correct: false },
      { text: "galvanometer", correct: false }
    ],
    answer: "altimeter"
  },
  {
    ques:
      "The Gate way of India is?",
    options: [
      { text: "Chennai", correct: false },
      { text: "Mumbai", correct: true },
      { text: "Kolkata", correct: false },
      { text: "New Delhi", correct: false }
    ],
    answer: "Mumbai"
  },
  {
    ques:
      "The chief ore of Aluminium is?",
    options: [
      { text: "Iron", correct: false },
      { text: "Cryolite", correct: false },
      { text: "Bauxite", correct: true },
      { text: "Haematite", correct: false }
    ],
    answer: "Bauxite"
  },
];

export default {
  name: "home",
  data() {
    return {
      presenceid: null,
      idToConnect: null,
      channel: null,
      hasEnded: false,

      hasAnswered: false,
      currentQuestionIndex: 0,
      question: null,
      options: null,
      show: false,
      correctanswer: null,
      
      timeLeft: 10,
      count: null,
      countdown: null,

      players: 1,
      secondplayer: false,
      playerdata: {
        one: {
          id: null,
          score: 0,
          userid: null
        },
        two: {
          id: null,
          score: 0,
          userid: null
        }
      },
      userid: null,
      url: null
    };
  },
  created() {
    this.currentQuestionIndex = 0;
    this.hasEnded = false;
    this.fetchData();
  },
  methods: {
    fetchData() {
      // Sets the data instance presenceid variable to the result of the getUniqueId function
      this.presenceid = this.getUniqueId();
      // This checks if there's no presence ID in the URL via the checkPresenceID function and appends the presenceid to the current URL so that we can have the URL end with a parameter like this https://hamilton-questions.firebaseapp.com/#/?id=agbew0gz
      if (!this.checkPresenceID()) {
        let separator = window.location.href.indexOf("?") === -1 ? "?" : "&";
        window.location.href =
          window.location.href + separator + this.presenceid;
      }
      // Sets the data instance url variable to the current URL.
      this.url = window.location.href;

      this.loadQuestion(this.currentQuestionIndex);
      //Initialize channel with current presenceid
      this.connectToChannel(this.idToConnect);
    },

    connectToChannel(presenceid) {
      // The channel variable is set to to the subscribeToPusher function in ChannelDetails.
      this.channel = ChannelDetails.subscribeToPusher(presenceid);
      this.defineChannel();
    },

    defineChannel() {
      let channel = this.channel;
      // The pusher:member_added event is triggered when a user joins a channel. We increase the number of players by one and also set the secondplayer boolean to true.
      channel.bind("pusher:member_added", members => {
        console.log("members", members);
        this.players += 1;
        this.secondplayer = true;
      });
      // Once a subscription has been made to a presence channel, an event is triggered with a members iterator.
      channel.bind("pusher:subscription_succeeded", members => {
        console.log("subscription_succeeded", members);
        // This checks if its just one player online and sets them up as player one and the required info for the game
        this.playerdata.one.userid = 1;
        this.playerdata.two.userid = 2;
        if (members.count === 1 && !this.playerdata.one.id) {
          this.playerdata.one.id = members.myID;
          this.userid = 1;
          // This checks if there's a player online already and sets the new player as player two.
        } else if (members.count === 2) {
          this.secondplayer = true;
          this.playerdata.two.id = members.myID;
          this.userid = 2;
        }
      });
      // The pusher:member_removed is triggered when a user leaves a channel. We decrease the number of players by one and also set the secondplayer boolean to false.
      channel.bind("pusher:member_removed", member => {
        console.log("member", member);
        this.players -= 1;
        this.secondplayer = false;
      });
      // This function receives new data from Pusher and updates the exisiting scores. This is what updates each player's score in realtime.
      channel.bind("client-send", data => {
        console.log("data", data);
        if (this.userid === 1) {
          this.playerdata.two.score = data.data.two.score;
        } else if (this.userid === 2) {
          this.playerdata.one.score = data.data.one.score;
        }
      });
      channel.bind("client-has-answered", data => {});

    },

    getUniqueId() {
      return (
        "id=" + (Math.floor(Math.random() * 10000000000 + 100000) % 500000)
        // Math.random().toString(36).substr(2, 8)
      );
    },

    checkPresenceID() {
      let getQueryString = (field, url) => {
        let href = url ? url : window.location.href;
        let reg = new RegExp("[?&]" + field + "=([^&#]*)", "i");
        let string = reg.exec(href);
        return string ? string[1] : null;
      };
      let id = getQueryString("id");
      return id;
    },

    checkAnswer(item) {
      let channel = this.channel;
      this.hasAnswered = true;
      clearInterval(this.countdown);

      if (item.text === this.correctanswer) {
        if (this.userid === 1) {
          this.playerdata.one.score += this.timeLeft;
        } else if (this.userid === 2) {
          this.playerdata.two.score += this.timeLeft;
        }
      }
      channel.trigger("client-send", {
        data: this.playerdata
      });
      channel.trigger("client-has-answered", {
        hasAnswered: true
      });

      this.count = 3;
      let countdown = setInterval(() => {
        this.count -= 1;
        if (this.count === 0) {
          clearInterval(countdown);
          this.currentQuestionIndex++;
          this.loadQuestion(this.currentQuestionIndex);
          this.show = false;
          this.showOptions();
        }
      }, 1000);
    },

    startTimer() {
        let channel = this.channel;

        this.timeLeft = 10;
        this.countdown = setInterval(() => {
          this.timeLeft--;

          if (this.timeLeft == 0) {
            clearInterval(this.countdown);
            this.timeOut();
          }
        }, 1000);
    },

    timeOut() {
      let channel = this.channel;
      this.hasAnswered = true;
      channel.trigger("client-has-answered", {
        hasAnswered: true
      });

      this.count = 3;
      this.countdown = setInterval(() => {
        this.count -= 1;
        if (this.count === 0) {
          clearInterval(this.countdown);
          this.currentQuestionIndex++;
          this.loadQuestion(this.currentQuestionIndex);
          this.show = false;
          this.showOptions();
        }
      }, 1000);
    },

    // getNewQuestion() {
    //   this.hasAnswered = false;
    //   this.currentQuestionIndex++;
    //   this.loadQuestion();

    //   if(this.currentQuestionIndex == questions.length) {
    //     this.endQuiz();
    //   }
    // },

    loadQuestion(index) {
      this.hasAnswered = false;

      if (index == questions.length) {
        this.endQuiz();
      } else {
        let question = questions[index];
        this.question = question;
        this.options = question.options;
        this.correctanswer = question.answer;
      }
    },

    showOptions() {
      setTimeout(() => {
        this.show = true;
        this.startTimer();
      }, 1000);

    },

    endQuiz() {
      this.hasEnded = true;
    }
  },
  watch: {
    secondplayer: function(val) {
      this.showOptions();
    }
  },
  computed: {
    idToShow() {
      return this.presenceid.substring(3, this.presenceid.length);
    },
    winner() {
      if (this.playerdata.one.score > this.playerdata.two.score) {
        return this.playerdata.one;
      } else {
        return this.playerdata.two;
      }
    }
  }
};
</script>


<style scoped>
.outer {
  min-height: 100%;
}
.home {
  display: flex;
  align-items: center;
  justify-content: center;
  height: fit-content;
}
.home input {
  font-size: 1.2em;
  box-shadow: 2px 5px 10px rgba(0, 0, 0, 0.2);
}
.home input::placeholder {
  color: rgb(200, 200, 200);
}
.home button {
  font-size: 1.6em;
  background-color: red;
  box-shadow: 2px 5px 10px rgba(0, 0, 0, 0.3);
}

.logo {
  animation: fade 0.8s ease-out forwards;
}
@keyframes fade {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
.idDiv {
  opacity: 0;
  animation: fade 0.8s ease-out forwards 0.3s;
}
.inputDiv {
  opacity: 0;
  animation: fade 0.8s ease-out forwards 0.5s;
}



.users-head {
  background: rgb(0, 0, 0, 0.4);
  /* background: linear-gradient(180deg, rgb(63, 0, 90) 0%, rgba(41, 7, 56, 0) 100%); */
  padding-top: 30px;
}
.user-img {
  height: 50px;
  width: 50px;
  background-color: white;
  margin: 10px auto;
}

.question {
  font-size: 2.5em;
  font-weight: 100;
  opacity: 0;
  transition: font-size ease-in 0.3s;
  animation: fade 0.8s ease-out forwards 0.2s;
}

.question-after {
  font-size: 2em;
}
.btn-option {
  min-width: 400px;
  background-color: rgba(255, 255, 255, 0.5);
  font-size: 1.2em;
  color: grey;
  transition: transform ease-in 0.1s;
}
.btn-option:hover {
  box-shadow: 2px 2px 20px rgba(255, 255, 255, 0.2);
  transform: translateY(-2px);
}

@media (max-width: 1024px) {
  .btn-option {
    min-width: 100%;
  }
}

.winners .user-img {
  height: 100px;
  width: 100px;
}

.winner {
  transform: scale(1.1);
  background-color: rgb(205, 235, 255);
  animation: winner 1.5s ease-in-out infinite;
}
@keyframes winner {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0;
}

.title {
  font-weight: bold;
}

.wronganswer {
  color: white;
  background-color: #f05757;
  opacity: 0.5;
  transition: background-color 0.5s ease;
}
.correctanswer {
  color: white;
  background-color: #00c4a7;
  transition: background-color 0.5s ease;
}
.wronganswer:hover,
.correctanswer:hover {
  color: white;
}
</style>