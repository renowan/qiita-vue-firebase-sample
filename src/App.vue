<template>
  <div id="app">

    <div class="container mt24">
      <div class="row">
        <div class="col-xs-12">
          <button type="button" class="btn btn-default" @click="login">
            匿名ユーザーでログイン
          </button>
        </div>

        <div class="col-xs-12 mt24">
          <button type="button" class="btn btn-default" @click="pushData">
            送信
          </button>
        </div>

      </div>
    </div>

  </div>
</template>

<script>
// データベースのインスタンス
const rootDb = firebase.database().ref('myBoard/')

export default {
  name: 'app',
  components: {

  },
  data () {
    return {
      list: []
    }
  },
  created () {
    firebase.auth().onAuthStateChanged( user => {
      if (user) {
        // User is signed in.
        console.log('is login.')
        this.listen()
      } else {
        // No user is signed in.
        console.log('No user is signed in.')
      }
    })
  },
  methods: {
    login () {
      firebase.auth().signInAnonymously().then(e => {
        console.log(e)
        this.listen()
      }).catch((error) => {
        // エラーメッセージ
        var errorCode = error.code;
        var errorMessage = error.message;
        console.log('エラーメッセージ', errorCode, errorMessage)
      });
    },
    // データベースの変更を購読、変更をlistに入れる
    listen () {
      firebase.database().ref('myBoard/').on('value', snapshot => {
        if (snapshot) {
          const rootList = snapshot.val()
          let list = []
          Object.keys(rootList).forEach((val, key) => {
            rootList[val].id = val
            list.push(rootList[val])
          })
          this.list = list
        }
      })
    },
    pushData () {
      rootDb.push({
        name: 'test',
        message: 'foo'
      })
    }
  }
}
</script>

<style>
.mt24 {
  margin-top: 24px;
}
</style>
