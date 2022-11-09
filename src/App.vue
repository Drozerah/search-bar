<template>
  <div id="app" class="container">
    <div class="debug" v-if="isDurty && isDebug">
      <div>Search Term: <strong>{{ previousSearch }}</strong></div>
      <div>Search Results: <strong>{{ data.length }}</strong></div>
      <div>Selected Items Number: <strong>{{ selection.length }}</strong></div>
      <div>Last Selection: <strong>{{ selected?.name || LastSelected?.name }}</strong></div>
    </div>
    <section>
        <b-field label="Find Vue.js based Github repositories">
            <b-autocomplete
                :data="data"
                placeholder="e.g. Component"
                field="title"
                icon="magnify"
                :loading="isFetching"
                v-model="currentSelect"
                :open-on-focus="openOnFocus"
                @typing="getAsyncData"
                @select="option => selected = option">
                <template slot-scope="props">
                  <div class="media">
                    <div class="media-content">
                        {{ props.option.name }}
                    </div>
                  </div>
                </template>
                <template #footer>
                  <div
                    v-if="previousSearch.length"
                    class="info">
                    <a @click="paginate()"><span>Add more results for <strong>'{{ previousSearch }}'</strong></span></a>
                  </div>
                </template>
            </b-autocomplete>
        </b-field>
        <div  class="cards">
          <div
            v-for="(item, idx) of selection"
            :key="`item_${idx}`"
            class="card"
            >
            <div class="card-content">
              <div class="content">
                <h3>{{ item.name }}</h3>
                <p>{{ item.description }}</p>
              </div>
            </div>
          </div>
        </div>
    </section>
  </div>
</template>

<script>
import { debounce } from 'underscore'
import { Octokit } from '@octokit/core'

export default {
  name: 'App',
  data () {
    return {
      data: [],
      selected: null,
      LastSelected: null,
      isFetching: false,
      selection: [],
      openOnFocus: false,
      currentSelect: '',
      previousSearch: '',
      isDurty: false,
      page: 1,
      isDebug: true
    }
  },
  methods: {
    getAsyncData: debounce(async function (name) {
      if (!name.length) {
        this.data = []
        return
      }
      const octokit = new Octokit({
        auth: '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'
      })
      try {
        this.isFetching = true
        const response = await octokit.request(`GET /search/repositories?page=${this.page}&per_page=10`, {
          q: `${name} in:file language:vue`,
          sort: 'stars',
          order: 'desc'
        })
        const { items } = response.data
        this.data = [...this.data, ...items]
        this.previousSearch = name
        this.openOnFocus = true
        this.isDurty = true
      } catch (error) {
        this.isFetching = false
        this.data = []
        throw error
      } finally {
        this.isFetching = false
      }
    }, 500),
    paginate: function () {
      if (this.previousSearch) {
        this.page++
        this.getAsyncData(this.previousSearch)
      }
    }
  },
  watch: {
    selected (val) {
      if (val) {
        this.selection.push(val)
        this.LastSelected = val
      }
    }
  }
}
</script>

<style lang="scss">

#app {
  margin: 50px auto 150px auto;
  .cards{
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  .info{
    text-align: center;
  }
  .label{
    font-size: 1.5rem;
  }
  .debug{
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    font-size: 12px;
    padding: 20px 50px;
    background-color: #efff02;
    z-index: 10;
  }
}
</style>
