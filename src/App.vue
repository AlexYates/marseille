<template>
  <main class="bg-white flex flex-col p-2">
    <Message v-if="hasError" v-html="'An error occured&hellip;'" />
    <Message v-if="!hasURLs" v-html="'No feed URLs found&hellip;'" />
    <Message v-if="isLoading" v-html="'Loading feeds&hellip;'" />
    <Message v-if="isProcessing" v-html="'Processing feeds&hellip;'" />
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
      ]
    };
  },
  created() {
    this.hasError = false;
    this.hasURLs = false;
    this.isLoading = false;
    this.isProcessing = false;
    if (this.urls.length > 0) {
      this.hasURLs = true;
      const collectAll = new Array(this.urls.length).fill(null).map((v, i) => {
        const url = this.urls[i];
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
};
</script>
