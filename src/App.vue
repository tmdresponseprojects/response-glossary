<template>
<form v-cloak>
<div id="app">
  <div class="panel panel-default">
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
    <p class="displayindex animated fadeInUp" v-on:click="showindex(), hidethelist()" v-bind:class="{hidden: indexbtnhide}">Click to view Index</p>
    <p class="closeindex animated fadeInUp" v-on:click="dontshowindex(), showthelist()" v-bind:class="{ hidden: hideindex }">Close</p>
    <div class = "index animated zoomIn" v-bind:class="{ hidden: hideindex }">
      <center>
        <p>Index</p>
      </center>
      <ul class="liindex">
        <li class="clickable" v-for="word in sortedWords" v-on:click="clicksearch(word), dontshowindex(), showthelist()">
          {{word.word}}
        </li>
      </ul>
    </div>
    </br>
    </br>
    <div class="outputdisplay">
      <ul class="listDisplay" v-bind:class="{hidden: listhide}">
        <li v-for="word in filteredWords">
          <p class="word">{{word.word}}</p>
            <div v-for="each in word.url">
              <a class="website" v-bind:href="each">Wikipedia Reference</a>
            </div>
          <p class="definition">{{word.definition}}</p>
        </li>
      </ul>
    </div>
	</div>
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
      console.log(wordsArray)
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
      console.log(wordsArray)
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
  p.definition{
    text-align: right;
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
    list-style-type: none
  }
  p.displayindex{
    text-align: right;
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
    text-align: right;
    font-family: cursive;
    font-size: 28px;
    font-weight: 400;
    line-height: 20px;
  }
  .hidden{
    display: none;
  }
</style>
