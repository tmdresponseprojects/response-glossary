<template>
<form v-cloak>
<div id="app">
  <div class="panel panel-default">
  	<div class="panel-body">
      <center>
        <div class="row">
          <div class="two columns">Glossary</div>
          <div class="eight columns">
            <input type="text" v-model="searchString" placeholder="Search" />
          </div>
          <div class="two columns">
            <div id="btnLogout" class="logout" v-on:click="LogOut()">
              <a class="sign">Log Out</a>
            </div>
            <div id="btnLogin" v-on:click="googleLogin()">
              <a class="sign">Google Sign In</a>
            </div>
          </div>
        </div>
      </center>
    </div>
  </div>
  <div id="bodyblock" class="container">
    <div class="row">
      <div class="three columns">
        <div class = "index">
          <center>
            <p>Index</p>
          </center>
          <ul class="liindex">
            <li class="clickable" v-for="word in sortedWords" v-on:click="clicksearch(word)">
              <center>
                {{word.word}}
              </center>
            </li>
          </ul>
        </div>
      </div>
      <div class="nine columns">
        <div class="outputdisplay">
          <ul class="listDisplay">
            <li v-for="word in filteredWords">
              <p class="word">{{word.word}}</p>
              <p class="definition">({{word.pos}}); {{word.definition}}</p>
              <p class="example">{{word.example}}</p>
            </li>
          </ul>
        </div>
      </div>
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
      searchString: ''
    }
  },
  firebase: {
    Words: WordsRef
  },
  computed: {
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
    clicksearch: function (w) {
      this.searchString = w.word
    },
    googleLogin: function () {
      Firebase.auth().signInWithPopup(provider).then(function (result) {
        console.log(result)
      })
    },
    LogOut: function () {
      Firebase.auth().signOut()
      document.getElementById('btnLogin').style.display = ''
    }
  }
}
Firebase.auth().onAuthStateChanged(firebaseUser => {
  if (firebaseUser) {
    firebaseUser.sendEmailVerification().then(function () {
    },
  )
    if (firebaseUser.emailVerified === true) {
      document.getElementById('btnLogout').style.display = ''
      document.getElementById('btnLogin').style.display = 'none'
      document.getElementById('bodyblock').style.display = ''
    } else {
      document.getElementById('btnLogout').style.display = 'none'
      document.getElementById('bodyblock').style.display = 'none'
    }
  } else {
    document.getElementById('btnLogout').style.display = 'none'
    document.getElementById('bodyblock').style.display = 'none'
  }
})
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
    height: 250px;
    overflow: scroll;
    border-radius: 5px;
    border-color: #2c3e50;
    background: grey;
  }
  div.outputdisplay {
     word-wrap: break-word;
  }
  ul.listDisplay {
    list-style-type: none;
  }
  p.word{
    font-size: 40px;
    font-weight: bold;
    font-variant: small-caps;
  }
  p.definition{
    text-indent: 1em;
  }
  p.example{
    text-indent: 1em;
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
</style>
