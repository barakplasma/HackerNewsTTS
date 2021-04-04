<template>
  <div id="app">
    <h1>{{ message }}</h1>
    <dl>
      <dt v-for="post in stories">
        <a :href="post.href" target="_blank">{{ post.title }}</a>
      </dt>
    </dl>
    <select v-model="index">
      <option v-for="(voice, index) in voices" :key="index" :value="index">
        {{ voice.name }}
      </option>
    </select>
    <button @click="readPage">Read top headlines</button>
    <button @click="stopReading">⏹</button>
  </div>
</template>

<script>
export default {
  data() {
    this.voices = window.speechSynthesis.getVoices();
    let daniel = this.voices.findIndex((v) => v.name === "Daniel");

    return {
      message: "HackerNews TTS",
      stories: this.stories,
      voices: this.voices,
      voice: this.voices[this.index],
      index: daniel > -1 ? daniel : 0
    };
  },
  methods: {
    stopReading() {
      window.speechSynthesis.pause();
    },
    readPage() {
      let utterance = new SpeechSynthesisUtterance(
        `...Top Hacker News Stories, narrated by Hacker News Text to speech; using the '${
          this.voices[this.index].name
        } voice: ...${this.stories
          .map((story) => story.title)
          .join(". Next story: ")}. End of stories`
      );
      utterance.voice = this.voices[this.index];
      speechSynthesis.speak(utterance);
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
}

button {
  background: none;
  border: solid 1px;
  border-radius: 2em;
  font: inherit;
  padding: 0.75em 2em;
}
</style>
