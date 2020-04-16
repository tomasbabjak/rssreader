<template>
  <div>
    <a :href="article.link" target="_blank">
      <b-card
        :title= this.article.title
        :sub-title = this.article.content
        :footer = this.getHost()
        :img-src= this.getImg()
        img-alt="Image"
        img-top
      >
      </b-card>
    </a>
  </div>
</template>

<script>
export default {
  name: "Article",
  props: ["article"],
  methods: {
    getHost() {
      if (this.article) {
        try {
          const url = new URL(this.article.link);
          return url.hostname;
        } catch (e) {
          console.error(e.toString());
        }
      }
    },
    getDate() {
      if (this.article) {
        return this.article.isoDate;
      }
    },
    getImg() {
      if (this.article) {
        let str = this.article.enclosure.url
        if (str.indexOf('.jpg') != -1 ) 
          str = str.slice(0, str.indexOf('.jpg') + 4)
        else if (str.indexOf('.png') != -1 ) 
          str= str.slice(0, str.indexOf('.png') + 4)
        console.log(str)
        return str
      }
    }
  }
};
</script>

<style scoped>
</style>
