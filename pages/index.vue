<template>
  <section id="main">
    <!-- データの入力 -->
    <div v-if = "playing">you are {{myteban}} now {{games[0].teban}}</div>
    <svg v-if = "playing" height = 200 width = 600>
      <circle  v-for="(koma,i) in JSON.parse(games[0].ban)"
      :cx = "i*100+50" cy = 100 r = 50 :fill="color(koma)"
      @click =  "move(i)">

    </circle>

  </svg>
  <div>
    <!-- {{ban}} -->
  </div>
  <v-btn @click = "start">start</v-btn>
</section>

</template>

<script>
import axios from 'axios';
import firebase from "~/plugins/firebase.js";
import { mapGetters} from 'vuex';
const db = firebase.firestore();

export default {
  data() {
    return {
      playing :false,
      //自分の番号
      myteban : null

    }
  },
  //データ　メソッドの中間
  computed: {
    // VuexからPostsデータを取得
    ...mapGetters(['games']),
    //gameというdocumentへの引用
    ban(){
      let ref = db.collection("games").doc("game");
      //documentをゲットすることができたらthenのなかのfunctionを実行
      ref.get().then(function(doc){
        //doc　ref.get()での関数の引数が入ってくる
        //文字列を配列に直す
        return JSON.parse(doc.data().ban);
      })
    }
  },
  created: function () {
    // firestoreのpostsをバインド database tb
    this.$store.dispatch('setGamesRef', db.collection('games'))
  },
  methods: {
    start :function () {
      const watashi = this;
      console.log(this.games[0].ban);
      this.playing = true;

      //手番を決める
      //dbに接続
      let ref = db.collection("games").doc("game");
      //documentをゲットすることができたらthenのなかのfunctionを実行
      //読み込み
      ref.get().then(function(doc){
        //自分がなんばんめのplayerか
        //今0人だったら1 , 1だったら２
        watashi.myteban = doc.data().num + 1;
        //dbを書き換える
        ref.update({num:watashi.myteban});
      })
    },
    color :function (d) {
      d = 1 * d;
      if (d == 1) {
        return "blue";

      } else if(d == 2) {
        return "red";

      }else{
        return "white";
      }

    },
    move :function (i) {
      //押した時に動けるか否か
      //myteban db tebanが一致した時ok
      const watashi = this;
      let ref = db.collection("games").doc("game");
      //documentをゲットすることができたらthenのなかのfunctionを実行
      ref.get().then(function(doc){

        if(watashi.myteban != doc.data().teban)return;

        //doc　ref.get()での関数の引数が入ってくる
        //文字列を配列に直す
        let conf =  JSON.parse(doc.data().ban);
        if(conf[i] != watashi.myteban ) return;

        if (watashi.myteban == 1) {

          if(conf[i+1]==0){

            let buf = conf[i];

            conf[i] = conf[i+1];

            conf[i+1] = buf;
          } else if (conf[i+1]==2 && conf[i+2]==0) {
              let buf = conf[i];

              conf[i] = conf[i+2];

              conf[i+2] = buf;


            }
          }else if (watashi.myteban == 2){
            if(conf[i-1]==0){

              let buf = conf[i];

              conf[i] = conf[i-1];

              conf[i-1] = buf;}
              else if (conf[i-1]==1 && conf[i-2]==0) {
                let buf = conf[i];

                conf[i] = conf[i-2];

                conf[i-2] = buf;


              }

            }

            //これをdbに書き込む

            console.log(conf);
            //
            ref.update({ban:JSON.stringify(conf),teban:3-doc.data().teban});


          })

        },
        sendData: function () {
          // データのチェック
          if (this.message == "" || this.message.length > 100) {
            return false;
          }
          let dbdata = {
            gData: this.message
          };
          // データの登録
          db.collection('games').add(dbdata);
        }
      }
    }
    </script>
