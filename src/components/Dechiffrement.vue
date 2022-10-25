<template>
  <div>
    <article>
      <div class="base-image-input" @click="$refs.fileInput.click()">
        <div class="overlay" :style="{ 'background-image': `url(${imageData})` }"></div>
        <span class="placeholder">
          <template v-if="!image">Choisir une image</template>
        </span>
        <input class="file-input" ref="fileInput" type="file" accept="image/*" @change="onSelectFile">
      </div>
    </article>
    <button class="process-button" :style="{ opacity: image ? '1' : '0.5' }" @click="recognize">Process</button>
    <p>{{status}}</p>
    <div class="progress-bar-container">
      <div class="progress-bar" :style="{ width: (this.progress * 100) + '%' }">
        <span class="progress-text">{{(progress * 100).toFixed(0)}}%</span>
      </div>
    </div>
    <div class="result">
      {{result}}
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import { createWorker, PSM, OEM } from 'tesseract.js';
import { ASSOCATION_TABLE } from '../data'
const levenshteinDistance = (str1 = '', str2 = '') => {
   const track = Array(str2.length + 1).fill(null).map(() =>
   Array(str1.length + 1).fill(null));
   for (let i = 0; i <= str1.length; i += 1) {
      track[0][i] = i;
   }
   for (let j = 0; j <= str2.length; j += 1) {
      track[j][0] = j;
   }
   for (let j = 1; j <= str2.length; j += 1) {
      for (let i = 1; i <= str1.length; i += 1) {
         const indicator = str1[i - 1] === str2[j - 1] ? 0 : 1;
         track[j][i] = Math.min(
            track[j][i - 1] + 1, // deletion
            track[j - 1][i] + 1, // insertion
            track[j - 1][i - 1] + indicator, // substitution
         );
      }
   }
   return track[str2.length][str1.length];
};

export default {
  name: 'app',
  data: () => ({
    image: null,
    imageData: null,
    status: '',
    progress: 0,
    worker: null,
    result: ''
  }),
  mounted: function () {
    this.worker = createWorker({
      logger: m => {
        this.status = m.status;
        this.progress = m.status == 'recognizing text' ? m.progress : 0
        console.log(m)
      }
    })
  },
  methods: {
    async recognize() {
      if (this.image == null) return;
      await this.worker.load();
      await this.worker.loadLanguage('fra');
      await this.worker.initialize('fra', OEM.LSTM_ONLY);
      await this.worker.setParameters({
        tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
      });
      const { data: { text } } = await this.worker.recognize(this.image);
      this.status = 'déchiffrement'
      let resultText = text
      const words = resultText.split(/[\s,.]/)
      console.log(words)
      words.forEach((word) => {
        if (word.length <= 2) return;
        let minDistance = 10000;
        let bestMatch = null;
        let bestMatchIndex = null
        Object.values(ASSOCATION_TABLE).forEach((name, i) => {
          let d = levenshteinDistance(word, name)
          if (d < minDistance) {
            minDistance = d
            bestMatch = name
            bestMatchIndex = i
          }
        })
        console.log(word, bestMatch, minDistance)
        resultText = resultText.replaceAll(word, Object.keys(ASSOCATION_TABLE)[bestMatchIndex])

      })
      this.result = resultText
    },
    onSelectFile (e) {
      e.preventDefault();
      this.image = e.target.files[0];
      const reader = new FileReader
      reader.onload = e => {
          this.imageData = e.target.result
      }
      reader.readAsDataURL(this.image)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  padding: 0 20px;
  text-align: center;
}

.base-image-input {
  position: relative;
  cursor: pointer;
  background-size: cover;
  background-position: center center;
  width: 100%;
  height: 30vh;
  border: dotted 2px #777777;
  border-radius: 20px;
}
.placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 18px;
  margin-top: 33%;
}
.file-input {
  display: none;
}
.overlay {
  position: absolute;
  background-position: center;
  background-size: contain;
  background-repeat: no-repeat;
  width: 100%;
  height: 100%;
  top:0;
  left: 0;
  opacity: 0.75
}

.process-button {
  background: #285436;
  color: white;
  font-size: 18px;
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
  outline: none;
  margin: 20px ;
}
.progress-bar-container {
  width: 100%;
  height: 20px;
  border: solid 1px #285436;
  border-radius: 5px;
  text-align: left;
}
.progress-bar {
  background: #285436;
  border-radius: 5px;
  height: 100%;
  position: relative;
}
.progress-text {
  position: absolute;
  right: 0;
  margin-right: 5px;
  margin-top: 2px;
  font-size: 15px;
  font-weight: bold;
  color: white;
}
.result {
  margin-top: 20px;
  width: 100%;
  height: auto;
  min-height: 20vh;
  border: solid 1px black;
}
</style>
