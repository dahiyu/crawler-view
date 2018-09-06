<template>
  <v-ons-page>
    <v-ons-toolbar modifier="transparent"></v-ons-toolbar>
    <div class="header">
      <!-- <img src="../assets/onsenui-logo.png"> -->
    </div>

    <v-ons-list-title>Links</v-ons-list-title>
    <v-ons-list>
      <v-ons-list-item modifier="chevron" v-for="item in essentialLinks" @click="goTo(item.link)" :key="item.label">
        <div class="left"><v-ons-icon fixed-width :icon="item.icon"></v-ons-icon></div>
        <div class="center">{{ item.label }}</div>
      </v-ons-list-item>
    </v-ons-list>
  </v-ons-page>
</template>

<script>
export default {
  name: 'menu-page',
  data () {
    return {
      essentialLinks: [
        {
          label: 'test1',
          link: 'google.co.jp',
          icon: 'fa-book'
        },
        {
          label: 'test2',
          link: 'google.co.jp',
          icon: 'fa-commenting'
        },
        {
          label: 'test3',
          link: 'google.co.jp',
          icon: 'ion-chatboxes'
        }
      ]
    }
  },
  methods: {
    goTo (url) {
      window.open(url, '_blank')
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.header {
  text-align: center;
  margin-bottom: 20px;
}

img {
  max-width: 150px;
}

ons-list-title {
  text-transform: none;
}

ons-list-item {
  cursor: pointer;
}
</style>
