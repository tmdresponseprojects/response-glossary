<template>
<form v-cloak>
<div id="app">
  <div id="top" class="panel panel-default">
  	<div class="panel-body">
      <center>
        <div class="row animated fadeInUp">
          <div class="two columns">
            <div class="toptext">
              Glossary
            </div>
          </div>
          <div class="eight columns">
            <input class="searchstyle" type="text" v-model="searchString" placeholder="Search" v-on:click="clearsearch()" />
          </div>
          <div class="two columns">
            <div class="toptext">
              <div v-on:click="LogOut(), handledauth()" v-bind:class="{ hidden: hideinfo }">
                <a class="sign">Log Out</a>
              </div>
              <div v-on:click="googleLogin(), handledauth()" v-bind:class="{ hidden: hidelogin }">
                <a class="sign">Google Sign In</a>
              </div>
            </div>
          </div>
        </div>
      </center>
    </div>
  </div>
  <div class="animated fadeInUp" v-bind:class="{ hidden: hideinfo }">
    <div class="parent">
      <ul class="grid">
        <div class="child">
          <li v-for="word in sortedWords">
            <div v-if="letteraccess(word) != false">
              <a class="jump" v-bind:href="giveref()" v-on:click="azpagelayout()">{{giveanchor()}}</a>
            </div>
          </li>
        </div>
      </ul>
    </div>
    <ul class="listDisplay">
      <li v-for="word in filteredWords">
        <div v-if="letteraccess(word) != false">
          <div class="letterjump" v-bind:id="giveanchor()">{{giveanchor()}}</div>
        </div>
        <div class="card">
          <p class="word">{{word.word}}</p>
          <p class="definition">{{word.definition}}</p>
          <div v-for="each in word.url">
            <a class="website" v-bind:href="each" target="_blank">Reference Article</a>
          </div>
        </div>
      </li>
    </ul>
    <a class="totop" href="#top"><i class="up"></i></a>
	</div>
  <img src="./assets/response.png" class="logingraphic" v-bind:class="{hidden: hidelogin}">
</div>
</form>
</template>

<script>
import Firebase from 'firebase'
var config = {
  apiKey: 'AIzaSyBdn9EdDahpPYDx0rJxYWGwmn3DaB-zP30',
  authDomain: 'response-glossary.firebaseapp.com',
  databaseURL: 'https://response-glossary.firebaseio.com',
  projectId: 'response-glossary',
  storageBucket: 'response-glossary.appspot.com',
  messagingSenderId: '133393930480'
}
const app = Firebase.initializeApp(config)
var provider = new Firebase.auth.GoogleAuthProvider()
let db = app.database()
let WordsRef = db.ref('Words')
var letterindex = ''
export default {
  data () {
    return {
      searchString: '',
      hidelogin: false,
      hideinfo: true
    }
  },
  firebase: {
    Words: WordsRef
  },
  computed: {
    // Functionality to display filter displayed words as user enters characters
    filteredWords: function () {
      var wordsArray = this.Words
      var searchString = this.searchString
      if (!searchString) {
        return wordsArray
      }
      searchString = searchString.trim().toLowerCase()
      wordsArray = wordsArray.filter(function (item) {
        if (item.word.toLowerCase().indexOf(searchString) !== -1) {
          return item
        }
      })
      return wordsArray
    },
    // Functionality to sort database in abc order
    sortedWords: function () {
      function compare (a, b) {
        if (a.word < b.word) {
          return -1
        }
        if (a.word > b.word) {
          return 1
        }
        return 0
      }
      return this.Words.sort(compare)
    }
  },
  methods: {
    // Functionality to place anchor destinations throughout page
    letteraccess: function (current) {
      let currentfirst = ''
      currentfirst = current.word[0]
      if (currentfirst === letterindex) {
        return false
      }
      if (currentfirst !== letterindex) {
        letterindex = currentfirst
        return currentfirst
      }
    },
    // Returns current index to store as destination for anchor tag
    giveanchor: function () {
      return letterindex
    },
    giveref: function () {
      var direct = '#'
      return (direct += letterindex)
    },
    // Sets up page layout for an a-z click address
    azpagelayout: function () {
      this.clearsearch()
    },
    // The search string is cleared
    clearsearch: function () {
      this.searchString = ''
    },
    // Login functionality for signing in with a google account
    googleLogin: function () {
      Firebase.auth().signInWithPopup(provider).then(function (result) {
        console.log(result)
      })
    },
    // Log out functionality
    LogOut: function () {
      Firebase.auth().signOut()
      this.clearsearch()
    },
    // When run will display screen for logged in user
    displayforlogin: function () {
      this.hideinfo = false
      this.hidelogin = true
    },
    // When run will display screen for no logged in user
    displayforlogout: function () {
      this.hideinfo = true
      this.hidelogin = false
    },
    // Handles display for authenticated and no logged in user
    handledauth: function () {
      Firebase.auth().onAuthStateChanged(firebaseUser => {
        if (firebaseUser) {
          firebaseUser.sendEmailVerification().then(function () {
          },
        )
          if (firebaseUser.emailVerified === true) {
            this.displayforlogin()
          } else {
            this.displayforlogout()
          }
        } else {
          this.displayforlogout()
        }
      })
      return null
    }
  },
  created: function () {
    Firebase.auth().onAuthStateChanged(firebaseUser => {
      if (firebaseUser) {
        firebaseUser.sendEmailVerification().then(function () {
        },
      )
        if (firebaseUser.emailVerified === true) {
          this.displayforlogin()
        } else {
          this.displayforlogout()
        }
      } else {
        this.displayforlogout()
      }
    })
    return null
  }
}
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
  }
  div.letterjump {
    font-size: 50px;
    left: 10px;
    text-align: center;
    background-color: lightgrey;
    width: 60px;
    font-family: cursive;
    border-radius: 25px;
    position: relative;
  }
  div.parent {
    width: 100%;
    overflow-x: scroll;
    align-items: center;
    font-size: 28px;
    text-align: center;
  }
  div.child {
    width: 437px;
    display: block;
    margin: auto;
  }
  div.toptext{
    font-family: cursive;
    font-size: 28px;
    line-height: 20px;
  }
  div.card{
    margin-right:35px;
    margin-left: 35px;
    position: relative;
  }
  p.word{
    text-align: left;
    font-size: 30px;
    font-weight: bold;
    font-variant: small-caps;
  }
  p.definition{
    font-style: italic;
  }
  a.jump {
    font-family: cursive;
  }
  a.totop {
    position: fixed;
    right: 20px;
    bottom: 0px;
    font-size: 24px;
  }
  a.website{
    font-style: italic;
  }
  a.sign{
    color: #2c3e50;
  }
  i.up {
    border: solid lightgrey;
    border-width: 0 3px 3px 0;
    display: inline-block;
    padding: 10px;
    transform: rotate(-135deg);
    -webkit-transform: rotate(-135deg);
  }
  i.up:hover {
    border: solid grey;
    border-width: 0 3px 3px 0;
    padding: 12px;
  }
  ul.listDisplay {
    list-style-type: none;
  }
  input.searchstyle {
    font-family: cursive;
  }
  img.logingraphic {
    display: block;
    margin: auto;
  }
  .grid li {
    display: inline-block;
  }
  .hidden{
    display: none;
  }
</style>
