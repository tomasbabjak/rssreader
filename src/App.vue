<template>
  <b-container>
    <b-row align-v="center" align-h="center">
      <b-col cols='12' md='6'>
        <b-avatar text="RSS" size="4rem"></b-avatar>
        <b-avatar text="r"></b-avatar>
        <b-avatar text="e"></b-avatar>
        <b-avatar text="a"></b-avatar>
        <b-avatar text="d"></b-avatar>
        <b-avatar text="e"></b-avatar>
        <b-avatar text="r"></b-avatar>
      </b-col>
      <b-col cols='12' md='6'>
        <b-form
          @submit="onSubmit"
        >
          <b-form-group id="input-group" 
            label="Zadajte URL:" 
            label-for="input">
            <b-form-input
              id="input"
              v-model="rssFeedForm.url"
              placeholder="https://www.sme.sk/rss-title"
            ></b-form-input>
          </b-form-group>
          <b-button block type="submit" 
            >Potvrdit</b-button>
        </b-form>
      </b-col>
    </b-row>
    <b-row align-h="center">
      <b-col cols='12' md='6'>
        <div class="preview-vue-rss-feed">
          <div v-if="error" class="error">{{ error }}</div>
          <div v-else class="feed">
            <b-row align-v="center" align-h="center">
              <b-col cols='10' md='10'>
                <h1 v-if="name">{{ name }}</h1>
                <h1 v-else>{{ feed.title }}</h1>
              </b-col>
              <b-col cols='2' md='2'>
                <b-button v-on:click="getData">
                  Reload
                </b-button>
              </b-col>
            </b-row>

            <div v-if="loading">
              <b-spinner label="Loading..."></b-spinner>
            </div>
            <div class="articles-container">
              <b-pagination
                pills 
                align="center"
                v-model="currentPage"
                :total-rows="this.getRows()"
                :per-page="perPage"
                aria-controls="my-table"
              ></b-pagination>
              <b-table 
                selectable
                :select-mode="selectMode"
                @row-selected="onRowSelected"
                id="my-table"
                :items="this.feed.items"
                :per-page="perPage"
                :current-page="currentPage"
                :fields="fields"
                :sort-by.sync="sortBy"
                :sort-desc.sync="sortDesc"
                responsive="sm"
                small
              >      
              </b-table>
            </div>
          </div>
        </div>
      </b-col>
      <b-col cols='12' md='6'>
        <Article
          v-bind:article="selected[0]"
        />
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import RSSParser from "rss-parser";
import Article from "./components/NewArticle.vue";
import axios from 'axios';

export default {
  name: "Feed",
  components: {
    Article
  },
  props: {
    loadMore: Boolean
  },
  data() {
    return {
      perPage: 5,
      currentPage: 1,
      loading: true,
      error: "",
      feed: {},
      url: "",
      name: "",
      rssFeedForm: {
        url: ""
      },
      selectMode: 'single',
      selected: [],
      fields: [
        { key: 'title', sortable: true, label: "Titulok" },
        { key: 'date', sortable: true, label: "Datum" }
      ],
    };
  },
  created() {
    this.getData();
  },
  watch: {
    url() {
      this.getData();
    }
  },
  methods: {
    getRows(){
      if (this.feed.items) return this.feed.items.length
      else return 10000
    },
    onRowSelected(item) {
        this.selected = item
        window.scrollTo(0, document.body.scrollHeight || document.documentElement.scrollHeight);
    },
    returnArticles() {
      var result = this.feed.items.map(function(el) {
        var o = Object.assign({}, el);
        let date = new Date(el.isoDate);
        o.date = date;
        return o;
      })
      this.feed.items=result;
    },
    formSubmit(e) {
      e.preventDefault();
      let currentObj = this;
      this.axios.post('http://localhost:8000/yourPostApi', {
          name: this.name,
          description: this.description
      })
      .then(function (response) {
          currentObj.output = response.data;
      })
      .catch(function (error) {
          currentObj.output = error;
      });
    },

    async getData() {
      this.error = "";
      this.loading = true;
      this.feed = {};
      try {
        var urlka = ''
        if(this.url == '') urlka = 'https://www.sme.sk/rss-title'
        else urlka = this.url;
        axios.post(`http://webp.itprof.sk:8000/fetchurl`, 
        {
          'url': urlka,
        },
        {
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/rss+xml'
          }
        }
        ).then((response) => {
          const parser = new RSSParser();
          parser.parseString(response.data, (err, parsed) => {
            this.loading = false;
            this.feed = parsed;
            this.returnArticles();
            console.log(parsed)
          });
        }).catch(error => {
          console.log(error.response)
        });
      } catch (err) {
        this.loading = false;
        this.error = err.toString();
      }
    },
    getArticles() {
      if (this.feed.items) {
        return this.feed.items;
      }
    },
    onSubmit(evt){
      evt.preventDefault()
      this.url=this.rssFeedForm.url
      this.getData()
    },
  }
};
</script>

<style>
h1 {
  margin: 5px;
  font-size: 20px;
}
.feed {
  text-align: left;
}
a {
  color: #194853;
}

.error {
  color: red;
}

.page-item.active .page-link {  
    background-color: #6c757d !important;  
    border-color: #6c757d !important;  
    color: white !important
}

.page-item .page-link {
  color: #6c757d !important
}
</style>
