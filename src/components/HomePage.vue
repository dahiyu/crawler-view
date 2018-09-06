<template>
  <v-ons-page>
    <v-ons-toolbar class="home-toolbar">
      <div class="left">
        <v-ons-toolbar-button @click="$emit('toggleMenu')">
          <v-ons-icon icon="ion-navicon, material:md-menu"></v-ons-icon>
        </v-ons-toolbar-button>
      </div>
      <div class="center">{{ msg }}</div>
    </v-ons-toolbar>

    <v-ons-search-input placeholder="search"></v-ons-search-input>

    <v-ons-list>
      <v-ons-list-item v-for="(item, index) in results" :key="item.link">
        <div>
          <div class="list-item__subtitle">
            {{ item.no }} {{ item.date }} <v-ons-button v-if="item.existsResponse" @click="toggle(index, item.thread, item.no)">response</v-ons-button>
          </div>
          <div v-html="item.text" class="list-item__title"></div>

          <div v-if="item.showResponse">
            <v-ons-list>
              <v-ons-list-item v-for="(reply, index) in item.replies" :key="item.link">
                <div>
                  <div class="list-item__subtitle">{{ reply.no }} {{ reply.date }}</div>
                  <div v-html="reply.text" class="list-item__title"></div>
                </div>
              </v-ons-list-item>
            </v-ons-list>
          </div>
        </div>
      </v-ons-list-item>
    </v-ons-list>
  
  </v-ons-page>
</template>

<script>

import axios from 'axios'

export default {
  name: 'home',
  data () {
    return {
      results: [],
      msg: 'Browsing BBS'
    }
  },
  methods: {
    // レスポンス表示ボタン押下時の処理
    toggle (index, thread, no) {
      // 表示フラグの反転
      this.results[index].showResponse = !this.results[index].showResponse
      // 既に取得済みであれば、再取得しない
      if (this.results[index].replies.length != 0) return
      // レスポンスの取得
      this.recursiveSearchResponse(index, thread, no)
    },
    // レスポンスを再帰的に取得
    recursiveSearchResponse(index, thread, no) {
      this.searchResponse(thread, no)
      .then(response => {
        let twoch = response.data._embedded.twoch

        if (twoch === null || twoch === undefined) return

        for (let i=0; i<twoch.length; i++) {
          this.results[index].replies.push(twoch[i])
        }
        for (let i=0; i<twoch.length; i++) {
          this.recursiveSearchResponse(index, twoch[i].thread, twoch[i].no)
        }
      })
    },
    // レスポンスの検索
    async searchResponse(thread, no) {
      let noRegix = ",*^" + no + "$,*"
      noRegix = encodeURIComponent(noRegix);
      return await axios.get(process.env.API_ENDPOINT + "/twoch/search/findByThreadAndReplyDestinations?thread=" + thread + "&" + "replyDestinations=" + noRegix)
    }
  },
  mounted() {
    axios.get(process.env.API_ENDPOINT + "/twoch?size=5000&sort=date,desc")
    .then(response => {
      let twoch = response.data._embedded.twoch

      // レスポンスが存在する書き込みにレスポンス存在フラグを立てる
      for (let i=0; i<twoch.length; i++) {
        // 改行コードをHTMLタグに変換
        twoch[i].text = twoch[i].text.replace('\n', '<br>')
        // 初期化
        twoch[i].existsResponse = false
        twoch[i].showResponse = false
        twoch[i].replies = []
        for (let j=0; j<twoch.length; j++) {
          if (!twoch[j].isResponse || twoch[j].isResponse === 'false') continue
          let replayDestinations = twoch[j].replyDestinations.split(',')

          for (let k=0; k<replayDestinations.length; k++) {
            if (replayDestinations[k] === twoch[i].no) {
              twoch[i].existsResponse = true
            }
          }
        }
      }

      // レスポンスを削除する（表示するのはレスポンスのない書き込みのみ）
      twoch = twoch.filter(function(v, i) {
        return (!v.isResponse || v.isResponse === 'false');
      });

      this.results = twoch
    })
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

ons-search-input {
    display: initial;
    position: relative;
}

</style>
