<template>
  <div id="app">
    <h1>{{ message }}</h1>
    <label for="select">Available Voices</label>
    <select v-model="selectedVoice">
      <option
        v-for="voice in [...voices]"
        :key="voice.name"
        :value="voice.name"
      >
        {{ voice.name }}
      </option>
    </select>
    <button v-if="status !== 'playing'" @click="readPage">Read top headlines</button>
    <button v-if="status === 'playing'" @click="stopReading">⏹</button>
    <button v-if="status === 'paused'" @click="resumeReading">Resume 🔊</button>
    <article>
      <dl>
        <dt v-for="post in stories">
          <a :href="post.href" target="_blank">{{ post.title }}</a>
        </dt>
      </dl>
    </article>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: "HackerNews TTS",
      stories: [],
      status: "stopped",
      voices: new Set(),
      selectedVoice: "no voices available",
      utterance: null,
    };
  },
  methods: {
    getVoices() {
      let availableVoices = speechSynthesis.getVoices();
      if (availableVoices.length > 0) {
        for (let v of availableVoices) {
          this.voices.add(v);
        }
      } else {
        setTimeout(this.getVoices, 500)
      }
    },
    stopReading() {
      speechSynthesis.pause();
      this.status = "paused"
    },
    resumeReading() {
      speechSynthesis.resume();
      this.status = "playing"
    },
    readPage() {
      const utterance = new SpeechSynthesisUtterance(
        `...Top Hacker News Stories, narrated by Hacker News Text to speech; using the '${
          this.selectedVoice
        } voice: ...${this.stories
          .map((story) => story.title)
          .join(". Next story: ")}. End of stories`
      );

      this.status = "playing"
      speechSynthesis.speak(utterance);
      utterance.onend = () => {this.status === 'stopped'; console.info('stopped')}
      utterance.onerror = console.error
      this.utterance = utterance;
    },
    async fetchTitle(id) {
      return await fetch(
        `https://hacker-news.firebaseio.com/v0/item/${id}.json`
      )
        .then((res) => res.json())
        .then((story) => {
          return {
            title: story.title,
            id,
            href: `https://news.ycombinator.com/item?id=${id}`
          };
        });
    },
    async fetchStories() {
      let stories = await fetch(
        "https://hacker-news.firebaseio.com/v0/topstories.json?print=pretty"
      )
        .then((res) => res.json())
        .then((topStories) => {
          return topStories;
        });

      this.stories = await Promise.all(
        stories.slice(0, 9).map((story) => this.fetchTitle(story))
      );
      
      this.getVoices();
    }
  },
  mounted() {
    this.fetchStories();
  }
};
</script>

<!-- Use preprocessors via the lang attribute! e.g. <style lang="scss"> -->
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

a,
button {
  color: #4fc08d;
  pointer-style: cursor;
}

button {
  background: none;
  border: solid 1px;
  border-radius: 2em;
  font: inherit;
  padding: 0.75em 2em;
}
</style>
