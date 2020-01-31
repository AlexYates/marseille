<template>
  <div>
    <nav class="bg-gray-800">
      <button
        v-if="!completedURLs"
        @click="loadURL"
        v-html="'Click to load more&hellip;'"
        class="bg-gray-200 flex flex-col mb-0 p-4 bg-gray-800 text-lg text-white text-center w-full"
      />
      <Message v-if="completedURLs" v-html="'All URLs loaded&hellip;'" />
      <Message v-if="hasError" v-html="'An error occured&hellip;'" />
      <Message v-if="!hasURLs" v-html="'No feed URLs found&hellip;'" />
      <Message v-if="isLoading" v-html="'Loading feeds&hellip;'" />
      <Message v-if="isProcessing" v-html="'Processing feeds&hellip;'" />
    </nav>
    <main class="bg-white flex flex-col p-2">
      <ol v-if="!hasError && hasURLs && !isLoading && !isProcessing">
        <li v-for="(item, index) in items" :key="index">
          <FeedHeading
            v-if="!item.content"
            :href="item.link"
            v-text="item.title"
          />
          <FeedPostList :posts="item.items" />
        </li>
      </ol>
    </main>
  </div>
</template>

<script>
const Parser = require("rss-parser");
const parser = new Parser({
  maxRedirects: 2,
  timeout: 5000,
  xml2js: {
    strict: true
  }
});
const CORS_ANYWHERE = "https://cors-anywhere.herokuapp.com/";

const logError = err => {
  /* eslint-disable-next-line */
  console.error(err)
};
const logWarn = warn => {
  /* eslint-disable-next-line */
  console.warn(warn)
};
const createFeedError = url => ({
  link: url,
  title: `An error occurred when parsing URL: ${url}`,
  items: []
});

export default {
  components: {
    Message: () => import("@/components/Message.vue"),
    FeedHeading: () => import("@/components/FeedHeading.vue"),
    FeedPostList: () => import("@/components/FeedPostList.vue")
  },
  data() {
    return {
      completedURLs: false,
      hasError: false,
      hasURLs: false,
      isLoading: false,
      isProcessing: false,
      items: [],
      urls: [
        "http://feeds.bbci.co.uk/news/rss.xml",
        "https://www.reddit.com/r/news/.rss",
        "https://www.swellmagnet.com/feed",
        "https://www.techrepublic.com/rssfeeds/whitepapers/",
        "https://hackernoon.com/feed",
        "http://feeds.feedburner.com/surfertoday/surfing?format=xml",
        "https://www.techrepublic.com/rssfeeds/articles/"
      ],
      urlsCurrent: []
    };
  },
  created() {
    this.loadURL();
  },
  methods: {
    loadURL() {
      const next = this.urls.pop();
      if (!next) {
        this.completedURLs = true;
        return;
      }
      this.urlsCurrent.push(next);
      this.hasError = false;
      this.hasURLs = false;
      this.isLoading = false;
      this.isProcessing = false;
      if (this.urlsCurrent.length > 0) {
        this.hasURLs = true;
        const collectAll = new Array(this.urlsCurrent.length)
          .fill(null)
          .map((v, i) => {
            const url = this.urlsCurrent[i];
            return parser
              .parseURL(`${CORS_ANYWHERE}${url}`)
              .then((res, rej) => {
                if (res) {
                  return res;
                }
                if (rej) {
                  logWarn(rej);
                } else {
                  createFeedError(url);
                }
              })
              .catch(err => logError(err));
          });
        Promise.all(collectAll)
          .then((res, rej) => {
            if (res) {
              this.items = res;
            }
            if (rej) {
              logWarn(rej);
            }
          })
          .catch(err => logError(err));
      }
    }
  }
};
</script>

<style scoped>
nav {
  position: sticky;
  top: 0px;
}
</style>
