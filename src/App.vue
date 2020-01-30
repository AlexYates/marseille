<template>
  <ol class="bg-white flex flex-col p-2">
    <li v-if="hasError || !hasURLs || isLoading || isProcessing">
      <h2
        v-if="hasError"
        v-html="'An error occured&hellip;'"
        class="bg-gray-200 flex flex-col mb-2 p-4 rounded bg-gray-800 text-lg text-white"
      />
      <h2
        v-if="!hasURLs"
        v-html="'No feed URLs found&hellip;'"
        class="bg-gray-200 flex flex-col mb-2 p-4 rounded bg-gray-800 text-lg text-white"
      />
      <h2
        v-if="isLoading"
        v-html="'Loading feeds&hellip;'"
        class="bg-gray-200 flex flex-col mb-2 p-4 rounded bg-gray-800 text-lg text-white"
      />
      <h2
        v-if="isProcessing"
        v-html="'Processing feeds&hellip;'"
        class="bg-gray-200 flex flex-col mb-2 p-4 rounded bg-gray-800 text-lg text-white"
      />
    </li>
    <li v-else v-for="(item, index) in items" :key="index">
      <a
        v-if="!item.content"
        :href="item.link"
        rel="noreferrer"
        target="_blank"
        class="bg-gray-200 flex flex-col mb-2 p-4 rounded bg-gray-800 text-white focus:text-red-400 hover:text-red-400"
      >
        <h2 v-text="item.title" class="font-semibold text-lg text-right" />
      </a>
      <ol>
        <li v-for="(feed, jIndex) in item.items" :key="jIndex">
          <a
            :href="feed.link"
            rel="noreferrer"
            target="_blank"
            class="news bg-gray-200 flex flex-col mb-2 p-4 rounded focus:text-red-800 hover:text-red-800"
          >
            <h2 v-text="feed.title" class="font-semibold mb-2 text-lg" />
            <div
              v-html="feed.content || feed['content:encoded']"
              class="mb-4 text-gray-800 text-sm"
            />
            <p
              v-text="feed.isoDate || feed.date"
              class="text-red-800 text-right text-sm"
            />
          </a>
        </li>
      </ol>
    </li>
  </ol>
</template>

<script>
const Parser = require("rss-parser");
const parser = new Parser({
  headers: { "User-Agent": "Marseille" },
  maxRedirects: 2,
  timeout: 4500,
  xml2js: {
    strict: true
  }
});
const CORS_ANYWHERE = "https://cors-anywhere.herokuapp.com/";

const logDebug = err => {
  /* eslint-disable-next-line */
  console.debug(err)
};
const logError = err => {
  /* eslint-disable-next-line */
  console.error(err)
};
const logWarn = warn => {
  /* eslint-disable-next-line */
  console.warn(warn)
};
logDebug("hi");
logWarn("hi");
logError("hi");
const createFeedError = url => ({
  link: url,
  title: `An error occurred when parsing URL: ${url}`,
  items: []
});

export default {
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
