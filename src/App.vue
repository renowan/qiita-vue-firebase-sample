<template>
  <div id="app">

    <div class="container mt24">

      <div class="row">
        <div class="col-xs-12">
          <button type="button" class="btn btn-default" @click="login">
            匿名ユーザーでログイン
          </button>
          <button type="button" class="btn btn-default" @click="pushData">
            ダミー送信
          </button>
        </div>

      </div>

      <hr>

      <div class="row mt24">
        <!-- 入力欄 -->
        <div class="col-xs-4">
          <h3>入力</h3>
          <div class="form-group">
            <label for="nameInput">名前</label>
            <input type="txt" class="form-control" id="nameInput" v-model="name">
          </div>
          <div class="form-group">
            <label for="messageInput">メッセージ</label>
            <input type="txt" class="form-control" id="messageInput" v-model="message">
          </div>
          <button type="button" class="btn btn-default" @click="sendMessage">
            送信
          </button>
        </div>
        <!-- リスト -->
        <div class="col-xs-8">
          <h3>データリスト(list)</h3>
          <ul>
            <li v-for="item in list">{{item.name}} / {{item.message}}</li>
          </ul>
        </div>
      </div>


    </div>

  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      list: [],     // 最新状態はここにコピーされる
      name: '',     // 名前
      message: '',  // 送信メッセージ
    }
  },
  created () {
    // 認証チェック
    firebase.auth().onAuthStateChanged( user => {
      if (user) {
        console.log('ログイン状態.')
        this.listen()
      } else {
        console.log('ログインしていない状態')
      }
    })
  },
  methods: {
    // ログイン関数
    login () {
      firebase.auth().signInAnonymously().then(e => {
        console.log(e)
        this.listen()
      }).catch((error) => {
        // ログインのエラーメッセージ
        var errorCode = error.code;
        var errorMessage = error.message;
        console.log('ログインエラーメッセージ', errorCode, errorMessage)
      });
    },
    // データベースの変更を購読、最新状態をlistにコピーする
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
    sendMessage () {
      // 空欄の場合は実行しない
      if (!this.name || !this.message) return
      // 送信
      firebase.database().ref('myBoard/').push({
        name: this.name,
        message: this.message
      })
      // 送信後inputを空にする
      this.name = ''
      this.message = ''
    },
    // ダミー送信
    pushData () {
      firebase.database().ref('myBoard/').push({
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
