<template>
  <div>
    <b-alert variant="danger"
             dismissible
             :show="showNetworkErrorAlert"
             @dismissed="showNetworkErrorAlert=false">
      Network Error!
    </b-alert>
    <b-progress v-show="loading" :value="100" :max="100" animated class="mb-3"></b-progress>
    <b-row class="searchbox">
      <b-col sm="12"><b-form-input id="input-none" type="text" v-model="keyword" placeholder="Search User Name"></b-form-input></b-col>
    </b-row>
    <lazy-component class="row">
      <playerblock v-for="(player, key, index) in players" :key="index" :player="player" v-show="search(player)"></playerblock>
    </lazy-component>
    <hr/>
    <nyaa-footer :info="info"></nyaa-footer>
  </div>
</template>

<script>
import axios from 'axios';
import PlayerBlock from './PlayerBlock';
import Footer from './Footer';

export default {
  name: 'PlayerList',
  props: ['info'],
  data() {
    return {
      players: [],
      showNetworkErrorAlert: false,
      keyword: '',
      searchTimer: null,
      loading: true,
      timer: null,
    };
  },
  async mounted() {
    let data;
    try {
      data = await axios.get('/static/data/players.json');
    } catch (error) {
      this.showNetworkErrorAlert = true;
      return;
    }
    this.players = data.data;
    this.loading = false;
  },
  watch: {
    keyword: 'lazyload',
  },
  methods: {
    lazyload() {
      clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        this.$Lazyload.lazyLoadHandler();
      }, 200);
    },
    search(player) {
      if (this.keyword.length < 1) {
        return true;
      }
      const keyword = this.keyword.toLowerCase();
      if (player.uuid.indexOf(keyword) !== -1) {
        return true;
      }
      if (player.playername.toLowerCase().indexOf(keyword) !== -1) {
        return true;
      }
      return player.names.some((name) => {
        if (name.name.toLowerCase().indexOf(keyword) !== -1) {
          return true;
        }
        return false;
      });
    },
  },
  components: {
    playerblock: PlayerBlock,
    nyaaFooter: Footer,
  },
};
</script>

<style scoped>
.searchbox {
  margin-bottom: 16px;
}
</style>
