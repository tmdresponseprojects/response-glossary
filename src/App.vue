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
              <div id="btnLogout" class="logout" v-on:click="LogOut(), handledauth()">
                <a class="sign" v-bind:class="{ hidden: hideinfo }">Log Out</a>
              </div>
              <div id="btnLogin" v-on:click="googleLogin(), handledauth()" v-bind:class="{ hidden: hidelogin }">
                <a class="sign">Google Sign In</a>
              </div>
            </div>
          </div>
        </div>
      </center>
    </div>
  </div>
  <div id="bodyblock" class="container animated fadeInUp" v-bind:class="{ hidden: hideinfo }">
    <p class="displayindex animated fadeInUp" v-on:click="showindex(), hidethelist()" v-bind:class="{hidden: indexbtnhide}">View Index</p>
    <p class="closeindex animated fadeInUp" v-on:click="dontshowindex(), showthelist()" v-bind:class="{ hidden: hideindex }">Close Index</p>
    <div class="index animated zoomIn" v-bind:class="{ hidden: hideindex }">
        <p class="indexheader">Index</p>
      <ul class="liindex">
        <li class="clickable" v-for="word in sortedWords" v-on:click="clicksearch(word), dontshowindex(), showthelist()">
          {{word.word}}
        </li>
      </ul>
    </div>
    <div class="outputdisplay">
      <ul class="listDisplay" v-bind:class="{hidden: listhide}">
        <div class="alphaaddress">
          <div class="parent">
            <div class="child">
              <a class="jump" href="#." v-on:click="azpagelayout()">.</a>
              <a class="jump" href="#A" v-on:click="azpagelayout()">a</a>
              <a class="jump" href="#B" v-on:click="azpagelayout()">b</a>
              <a class="jump" href="#C" v-on:click="azpagelayout()">c</a>
              <a class="jump" href="#D" v-on:click="azpagelayout()">d</a>
              <a class="jump" href="#E" v-on:click="azpagelayout()">e</a>
              <a class="jump" href="#F" v-on:click="azpagelayout()">f</a>
              <a class="jump" href="#G" v-on:click="azpagelayout()">g</a>
              <a class="jump" href="#H" v-on:click="azpagelayout()">h</a>
              <a class="jump" href="#I" v-on:click="azpagelayout()">i</a>
              <a class="jump" href="#J" v-on:click="azpagelayout()">j</a>
              <a class="jump" href="#K" v-on:click="azpagelayout()">k</a>
              <a class="jump" href="#L" v-on:click="azpagelayout()">l</a>
              <a class="jump" href="#M" v-on:click="azpagelayout()">m</a>
              <a class="jump" href="#N" v-on:click="azpagelayout()">n</a>
              <a class="jump" href="#O" v-on:click="azpagelayout()">o</a>
              <a class="jump" href="#P" v-on:click="azpagelayout()">p</a>
              <a class="jump" href="#Q" v-on:click="azpagelayout()">q</a>
              <a class="jump" href="#R" v-on:click="azpagelayout()">r</a>
              <a class="jump" href="#S" v-on:click="azpagelayout()">s</a>
              <a class="jump" href="#T" v-on:click="azpagelayout()">t</a>
              <a class="jump" href="#U" v-on:click="azpagelayout()">u</a>
              <a class="jump" href="#V" v-on:click="azpagelayout()">v</a>
              <a class="jump" href="#W" v-on:click="azpagelayout()">w</a>
              <a class="jump" href="#X" v-on:click="azpagelayout()">x</a>
              <a class="jump" href="#Y" v-on:click="azpagelayout()">y</a>
              <a class="jump" href="#Z" v-on:click="azpagelayout()">z</a>
            </div>
          </div>
        </div>
        <li v-for="word in filteredWords">
          <div v-if="letteraccess(word) != false">
            <div class="letterjump" v-bind:id="giveanchor()">{{giveanchor()}}</div>
          </div>
          <p class="word">{{word.word}}</p>
          <p class="definition">{{word.definition}}</p>
          <div v-for="each in word.url">
            <a class="website" v-bind:href="each" target="_blank">Reference Article</a>
          </div>
        </li>
      </ul>
    </div>
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
      hideinfo: true,
      hideindex: true,
      indexbtnhide: false,
      listhide: false
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
    // Sets up page layout for an a-z click address
    azpagelayout: function () {
      this.clearsearch()
      this.dontshowindex()
      this.showthelist()
    },
    // The search strings text becomes the input 'w'
    clicksearch: function (w) {
      this.searchString = w.word
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
    // Functionaity to show index
    showindex: function () {
      this.hideindex = false
      this.indexbtnhide = true
    },
    // Functionality to hide index
    dontshowindex: function () {
      this.hideindex = true
      this.indexbtnhide = false
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
    // When called the list will show. Called after index is closed
    showthelist: function () {
      this.listhide = false
    },
    // When called the list will hide. Called when index is opened
    hidethelist: function () {
      this.listhide = true
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
  div.index {
    width: 100%;
    height: 100vh;
    overflow: scroll;
    border-radius: 5px;
    border-color: #2c3e50;
  }
  div.outputdisplay {
     word-wrap: break-word;
  }
  ul.listDisplay {
    list-style-type: none;
  }
  p.word{
    text-align: left;
    font-size: 40px;
    font-weight: bold;
    font-variant: small-caps;
  }
  p.indexheader{
    text-align: center;
  }
  p.definition{
    font-style: italic;
  }
  a.website{
    font-style: italic;
  }
  a.sign{
    color: #2c3e50;
  }
  li.clickable:hover{
    font-size: 24px;
  }
  li.clickable{
    font-size: 16px;
    line-height: 24px;
    transition: font 0.3s;
  }
  ul.liindex{
    list-style-type: none;
  }
  p.displayindex{
    text-align: center;
    font-family: cursive;
    font-size: 28px;
    font-weight: 400;
    line-height: 20px;
  }
  div.toptext{
    font-family: cursive;
    font-size: 28px;
    font-weight: 400;
    line-height: 20px;
  }
  input.searchstyle {
    font-family: cursive;
  }
  p.closeindex{
    text-align: center;
    font-family: cursive;
    font-size: 28px;
    font-weight: 400;
    line-height: 20px;
  }
  a.jump {
    display: inline;
    font-family: cursive;
  }
  div.letterjump {
    text-align: center;
    background-color: lightgrey;
    font-family: cursive;
  }
  div.alphaaddress {
      text-align: center;
      font-size: 24px;
  }
  div.parent {
    width: 100%;
    overflow-x: scroll;
    align-items: center;
  }
  div.child {
    width: 464px;
    display: block;
    margin: auto;
  }
  a.totop {
    position: fixed;
    right: 20px;
    bottom: 0px;
    font-size: 24px;
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
  img.logingraphic {
    display: block;
    margin: auto;
  }
  .hidden{
    display: none;
  }
</style>
