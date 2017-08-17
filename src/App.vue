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
            <input class="searchstyle" type="text" v-model="searchString" placeholder="Search"/>
            <input type="button" value="Clear" v-on:click="clearsearch()">
            <div class="parent" v-bind:class="{ hidden: hideinfo }">
              <ul class="grid">
                <div class="child">
                  <li v-for="letter in allpossible">
                    <a v-if="doesexist(letter)"class="jump" v-bind:href="giveref(letter)" v-on:click.prevent="()=>azpagelayout(letter)">{{letter}}</a>
                    <a v-else="doesexist(letter)"class="notactive" v-bind:href="giveref(letter)">{{letter}}</a>
                  </li>
                </div>
              </ul>
            </div>
          </div>
          <div class="two columns">
            <div class="toptext">
              <div v-on:click="LogOut(), handledauth()" v-bind:class="{ hidden: hideinfo }">
                <a>Log Out</a>
              </div>
              <div v-on:click="googleLogin(), handledauth()" v-bind:class="{ hidden: hidelogin }">
                <a>Google Sign In</a>
              </div>
            </div>
          </div>
        </div>
      </center>
    </div>
  </div>
  <div id="body" class="animated fadeInUp" v-bind:class="{ hidden: hideinfo }">
    <ul class="listDisplay">
      <li v-for="word in filteredWords">
        <div class="one column"v-if="letteraccess(word) != false">
            <div class="letterjump" v-bind:id="giveanchor()">{{giveanchor()}}</div>
        </div>
        <div class="card">
          <p class="word">{{word.word}}</p>
          <p class="definition">{{word.definition}}</p>
          <div v-for="each in word.url">
            <a class="website" v-bind:href="each" target="_blank">{{each}}</a>
          </div>
        </div>
      </li>
    </ul>
	</div>
  <div v-for="word in sortedWords"></div>
  <img src="./assets/response.png" class="logingraphic" v-bind:class="{hidden: hidelogin}">
  <a class="totop animated fadeIn" href="#app" v-bind:class="{hidden: hidetopjump}"><i class="up"></i></a>
</div>
</form>
</template>

<script>
import Firebase from 'firebase'
var config = {
  apiKey: 'AIzaSyBzXYBWtYP9Ns1hOfiu5dvi4ZqBIWJVk_I',
  authDomain: 'responseglossary.firebaseapp.com',
  databaseURL: 'https://responseglossary.firebaseio.com',
  projectId: 'responseglossary',
  storageBucket: 'responseglossary.appspot.com',
  messagingSenderId: '864755262659'
}
const app = Firebase.initializeApp(config)
var provider = new Firebase.auth.GoogleAuthProvider()
let db = app.database()
let WordsRef = db.ref('Words')
var letterindex = ''
var currentexists = []
export default {
  data () {
    return {
      hidelogin: false,
      hideinfo: true,
      scrolled: false,
      hidetopjump: true,
      searchString: '',
      allpossible: ['.', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R',
        'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z']
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
    // Handles when the jump to top button will show
    handleScroll: function () {
      this.scrolled = window.scrollY > 0
      this.hidetopjump = false
      if (window.scrollY === 0) {
        this.hidetopjump = true
      }
    },
    // Functionality to place anchor destinations throughout page
    letteraccess: function (current) {
      let currentfirst = ''
      currentfirst = current.word[0]
      if (currentfirst === letterindex) {
        return false
      }
      if (currentfirst !== letterindex) {
        letterindex = currentfirst
        if (currentexists.indexOf(currentfirst) <= -1) {
          currentexists.push(currentfirst)
        }
        return currentfirst
      }
    },
    // Checks to see if a "first letter" exists
    doesexist: function (input) {
      if (currentexists.indexOf(input) <= -1) {
        return false
      } else {
        return true
      }
    },
    // Returns current index to store as destination for anchor tag
    giveanchor: function () {
      return letterindex
    },
    giveref: function (letter) {
      var direct = '#'
      return (direct += letter)
    },
    // Sets up page layout for an a-z click address
    azpagelayout: function (letter) {
      this.clearsearch()
      var elem = document.getElementById(letter)
      var location = elem.getBoundingClientRect()
      window.scroll(0, location.top + window.scrollY - 170)
    },
    // The search string is cleared
    clearsearch: function () {
      this.searchString = ''
    },
    // Login functionality for signing in with a google account
    googleLogin: function () {
      Firebase.auth().signInWithPopup(provider).then(function (result) {
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
    window.addEventListener('scroll', this.handleScroll)
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
    overflow-x: hidden;
  }
  #top {
    position: fixed;
    width: 100%;
    z-index: 100;
    background-color: white;
    top: 0px;
  }
  #body {
    position: relative;
    padding-top: 170px;
  }
  div.letterjump {
    font-size: 50px;
    text-align: center;
    background-color: lightgrey;
    width: 60px;
    font-family: cursive;
    border-radius: 25px;
    position: relative;
    left: 20px;
  }
  div.parent {
    width: 100%;
    overflow-x: scroll;
    overflow-y: hidden;
    align-items: center;
    font-size: 28px;
    text-align: center;
    max-height: 39px;
  }
  div.child {
    width: 526px;
    display: block;
    margin: auto;
  }
  div.toptext {
    font-family: cursive;
    font-size: 28px;
    line-height: 20px;
  }
  div.card {
    margin-right:35px;
    margin-left: 100px;
    margin-bottom: 50px;
    position: relative;
  }
  p.word {
    text-align: left;
    font-size: 30px;
    margin-bottom: 10px;
    font-weight: bold;
    font-variant: small-caps;
  }
  p.definition {
    font-style: italic;
    margin-bottom: 10px;
    max-width: 600px;
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
  a.website {
    font-style: italic;
  }
  a.notactive {
   pointer-events: none;
   cursor: default;
   font-family: cursive;
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
    margin-top: 150px;
  }
  .grid li {
    display: inline-block;
  }
  .hidden {
    display: none;
  }
</style>
