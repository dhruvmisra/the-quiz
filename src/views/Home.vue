<template>
  <transition name="fade">
    <section class="outer" v-if="!hasEnded">
      <div class="home" v-if="!secondplayer">
        <div class="container">
          <h1 style="font-size: 10em">Q</h1>
          <h4 class="display-4">The Quiz</h4>

          <div class="row justify-content-around pt-5 my-5">
            <div class="mb-5">
              <p>Share this ID with your friend to connect</p>
              <h1 class="text center">{{ idToShow }}</h1>
            </div>

            <div>
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
          <p class="question" :class="{'question-after': hasAnswered }">{{question.lyric}}?</p>
          <div class="container mt-3">
            <div class="row justify-content-center mx-auto">
              <button
                class="btn btn-option m-2 w-25 p-5"
                :class="{ 'wronganswer': hasAnswered && !item.correct, 'correctanswer': hasAnswered && item.correct}"
                @click="checkAnswer(item)"
                v-for="item in options"
                :key="item.text"
              >{{item.text}}</button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <div v-else>over</div>
  </transition>
</template>

<script>
// Import ChannelDetails component created above
import ChannelDetails from "@/components/ChannelDetails";
// An array that holds the lyrics questions and their correct answers. All questions can be seen here https://gist.github.com/yomete/2d851c2adc008a9763a0db9f85879083
const lyrics = [
  {
    lyric:
      "When he was ten his father split, full of it, debt-ridden. Two years later, see Alex and his mother bed-ridden. Half-dead sittin' in their own sick, the scent thick",
    options: [
      { text: "Aaron Burr", correct: false },
      { text: "James Madison", correct: false },
      { text: "John Laurens", correct: false },
      { text: "Eliza Hamilton", correct: true }
    ],
    answer: "Eliza Hamilton"
  },
  {
    lyric:
      "I am sailing off to London. I’m accompanied by someone who always pays. I have found a wealthy husband. Who will keep me in comfort for all my days. He is not a lot of fun, but there’s no one",
    options: [
      { text: "Eliza", correct: false },
      { text: "Peggy", correct: false },
      { text: "Angelica", correct: true },
      { text: "Maria", correct: false }
    ],
    answer: "Angelica"
  }
];

export default {
  name: "home",
  data() {
    return {
      // This holds the current presence-id
      presenceid: null,
      idToConnect: null,
      channel: null,
      hasEnded: false,
      hasAnswered: false,
      currentQuestionIndex: 0,
      question: null,
      options: null,
      correctanswer: null,
      timeLeft: 10,
      // This is used for a countdown timer
      count: null,
      countdown: null,
      // Number of players in the game
      players: 1,
      // This checks if there's a second player, it becomes true when players = 2
      secondplayer: true,
      // This holds the player data for both players
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
      // This holds the userid for the current player
      userid: null,
      // This holds the current URL of the game
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
      // This checks if there's no presence ID in the URL via the checkPresenceID function and appends the presenceid to the current URL so that we can have the URL end with a parameter like this https://hamilton-lyrics.firebaseapp.com/#/?id=agbew0gz
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
        if (members.count === 1 && !this.playerdata.one.id) {
          this.playerdata.one.id = members.myID;
          this.playerdata.one.userid = 1;
          this.userid = 1;
          // This checks if there's a player online already and sets the new player as player two.
        } else if (members.count === 2) {
          this.secondplayer = true;
          this.playerdata.two.id = members.myID;
          this.playerdata.two.userid = 2;
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
      // channel.bind("client-question-update", data => {
      //   console.log('question', data, 'lyrics', lyrics.length);
      //   if(data.currentQuestionIndex == lyrics.length) {
      //     this.endQuiz();
      //   } else {
      //     this.currentQuestionIndex = data.currentQuestion;
      //     this.startTimer();
      //   }
      // });
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
          this.startTimer();
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
          this.startTimer();
        }
      }, 1000);
    },

    // getNewQuestion() {
    //   this.hasAnswered = false;
    //   this.currentQuestionIndex++;
    //   this.loadQuestion();

    //   if(this.currentQuestionIndex == lyrics.length) {
    //     this.endQuiz();
    //   }
    // },

    loadQuestion(index) {
      this.hasAnswered = false;

      if (index == lyrics.length) {
        this.endQuiz();
      } else {
        let question = lyrics[index];
        this.question = question;
        this.options = question.options;
        this.correctanswer = question.answer;
      }
    },

    endQuiz() {
      this.hasEnded = true;
    }
  },
  watch: {
    secondplayer: function(val) {
      this.startTimer();
    }
  },
  computed: {
    idToShow() {
      return this.presenceid.substring(3, this.presenceid.length);
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

.users-head {
  background: rgb(0, 0, 0, 0.4);
  /* background: linear-gradient(180deg, rgb(63, 0, 90) 0%, rgba(41, 7, 56, 0) 100%); */
  padding-top: 30px;
}
.user-img {
  height: 50px;
  width: 50px;
  margin: 10px auto;
}

.question {
  font-size: 2.5em;
  font-weight: 100;
  transition: font-size ease-in 0.3s;
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

.play--button {
  background-color: white;
  color: #7fd4d3;
  font-weight: bold;
  border-radius: 20px;
  letter-spacing: 1px;
  padding: 20px;
  transition: all 0.3s ease;
  text-shadow: 0 1px 3px rgba(36, 180, 126, 0.4);
  text-transform: uppercase;
  box-shadow: 0 4px 6px rgba(50, 50, 93, 0.11), 0 1px 3px rgba(0, 0, 0, 0.08);
}
.play--button:hover {
  background-color: white;
  color: #7fd4d3;
  transform: translateY(-1px);
  box-shadow: 0 7px 14px rgba(50, 50, 93, 0.1), 0 3px 6px rgba(0, 0, 0, 0.08);
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
.wronganswer:hover, .correctanswer:hover {
  color: white;
}
</style>