<template>
  <z-view style="border-width: 7px;">
    <div v-if="sharedState.axiosError !== ''">
        Oops!! {{sharedState.axiosError}}
      </div>
    <div v-if="search">
      <input type="text" placeholder="type a language ..." :value="language"
      @input="searchLanguages($event.target.value)">
    </div>
    <div slot="extension">
      <div v-if="query !== '' && search && $zircle.getCurrentViewName() === 'languages--0'">
      <z-spot

      v-for="(lang, index) in wt"
      button

      :key="'lang' + index"
      :distance="60"
      :angle="-90 + (360 / wt.length * index)"
      size="xs"
      class="test1 accent butt"
      :label="lang.name"
      @click.native="sharedState.language = lang.urlParam"
      style="border: none"
     :style="sharedState.language === lang.urlParam ? 'background-color:' + sharedState.colorMe.main : 'background-color: #454545; border: 1px solid ' + sharedState.colorMe.main"

      >
      </z-spot>
      </div>
      <div v-if="wt.length === 0 && search && searchActive">
      <z-spot
      button
      :distance="60"
      :angle="-90"
      size="xs"
      class="test1 accent butt"
      label="😕 try another term"
      style="border: none"
     :style="sharedState.language === language.urlParam ? 'background-color:' + sharedState.colorMe.main : 'background-color: #454545; border: 1px solid ' + sharedState.colorMe.main"

      >
      </z-spot>
      </div>

    <z-spot button
      class="buttons"
            size='s'
            :angle="45"
            :label="!search ? '+ languages' : 'go back'"
            :distance="130"
            @click.native="getLanguages()"
            @mouseup.native="search = !search"
            >
            <i v-if="!search" class="fas fa-search"></i>
            <i v-if="search" class="fas fa-undo"></i>
          </z-spot>
          <div v-if="popular.length && !search && $zircle.getCurrentViewName() === 'languages--0'">
     <z-list
        :items="popular"
        :per-page="8">
          <z-spot
           class="test1 butt"
            button
            size='xs'
            :distance='60'
            slot-scope="props"
            :index="props.index"
            :label="props.name"
            @click.native="sharedState.language = props.urlParam"
            style="border: none"
            :style="sharedState.language === props.urlParam ? 'background-color:' + sharedState.colorMe.main : 'background-color: transparent; border: 1px solid ' + sharedState.colorMe.main"
            >
          </z-spot>
      </z-list>
    </div>
      <z-spot  class="buttons" button size=xs :distance=125 :angle=-45 label="daily"
      style="border: none; color: white"
      :style="sharedState.since === 'daily' ? 'background-color: #5484f8' : ''"
      @click.native="sharedState.since = 'daily'" >T</z-spot>

      <z-spot  class="buttons" button size=xs :distance=125 :angle=-20 label="weekly"
      style="border: none; color: white"
      :style="sharedState.since === 'weekly' ? 'background-color: #5484f8' : ''"
      @click.native="sharedState.since = 'weekly'" >W</z-spot>

      <z-spot  class="buttons" button size=xs :distance=125 :angle=5 label="monthly"
      style="border: none; color: white"
      :style="sharedState.since === 'monthly' ? 'background-color: #5484f8' : ''"
      @click.native="sharedState.since = 'monthly'" >M</z-spot>
    </div>
  </z-view>
</template>
<script>
// :angle="(180 - (180 - ($zircle.getNumberOfPages() * 10))) / $zircle.getNumberOfPages() * ($zircle.getNumberOfPages() - index) + ((180 - (180 - (180 - ($zircle.getNumberOfPages() * 10)))) - ((180 - (180 - ($zircle.getNumberOfPages() * 10))) / $zircle.getNumberOfPages())) / 2"
import state from '../store/state'
import axios from 'axios'
function fetchGalleries (results, since, stateError) {
  return Promise.all(results.map(record => {
    return axios.get('https://github-trending-api.now.sh/repositories?since=' + since + '&language=' + encodeURIComponent(record.urlParam))
      .catch((err) => {
        console.log(err)
        stateError = err.message
      })
  })).then(gal => {
    var papa = gal.filter(function (el) {
      return el.data.length > 0
    })
    return papa.map(a => a.data[0].language)
  })
}
export default {
  data () {
    return {
      time: false,
      popular: [],
      other: [],
      results: [],
      query: '',
      wt: [],
      search: false,
      sharedState: state.$data,
      searchActive: false
    }
  },
  computed: {
    viewn () {
      return this.$zircle.getCurrentViewName()
    },
    language () {
      return this.query
    }
  },
  watch: {
    viewn: function () {
      // if (this.$zircle.getCurrentViewName() === 'repos--0') {
      this.popular = []
      this.results = []
      // }
    }
  },
  methods: {
    searchLanguages (e) {
      var vm = this
      if (e !== '') {
        var input = e.toLowerCase()
        var preResults = this.other.filter(function (el) {
          var data = el.name.toLowerCase()
          return data.indexOf(input) > -1
        })
        var sorted = preResults.sort(function (a, b) {
          return a.name.length - b.name.length
        })
        this.results = sorted.slice(0, 7)
        var pap = fetchGalleries(vm.results, vm.sharedState.since, vm.sharedState.axiosError)
        pap.then(result => {
          vm.wt = result.map(a => {
            var url = a.replace(/\s+/g, '-').toLowerCase()
            return {
              name: a,
              urlParam: url
            }
          })
          vm.searchActive = true
        })
        this.query = e
      } else {
        this.searchActive = false
        this.query = ''
        this.results = []
        this.wt = []
      }
    },
    getLanguages () {
      var vm = this
      this.popular = []
      this.query = ''
      this.results = []
      axios
        .get('https://github-trending-api.now.sh/languages')
        .then(function (response) {
          var res = response.data.popular
          res.push({ name: 'all languages', urlParam: '' })
          var lang = ''
          vm.sharedState.language === '' ? lang = 'all languages' : lang = vm.sharedState.language
          var checkLangSelected = res.filter(e => e.name.toLowerCase() === lang.toLowerCase())
          if (checkLangSelected.length) {
            vm.popular = res
          } else {
            res.push({ name: vm.sharedState.language, urlParam: vm.sharedState.language.replace(/\s+/g, '-').toLowerCase() })
            vm.popular = res.slice(-8)
          }
          vm.other = response.data.all
        })
        .catch((err) => {
          console.log(err)
          this.sharedState.axiosError = err.message
        })
    }
  },
  mounted () {
    this.sharedState.axiosError = ''
    this.getLanguages()
    this.sharedState.clearResults = true
  }
}
</script>
<style>
.test1>.z-label.bottom>.inside {
  background-color: transparent;
  font-size: 13px !important;
  top: 105% !important;
  color: white
}
</style>
