<template>
  <div class="p-6 mt-12 w-1/2 mx-auto">
    <div v-if="ready" class="flex flex-col items-start space-y-4">
      <video v-if="video" :src="videoSrc" controls width="250"></video>
      <input type="file" @change="(e) => (video = e.target.files[0])" />
      <button :disabled="!video" @click="convertToGif">Convert</button>
      <img v-if="gif" :src="gif" width="250" />
    </div>
    <p v-else>Loading...</p>
  </div>
</template>

<script>
import { createFFmpeg, fetchFile } from '@ffmpeg/ffmpeg'
const ffmpeg = createFFmpeg({ log: true })

export default {
  data() {
    return {
      ready: false,
      video: null,
      gif: null
    }
  },

  async mounted() {
    await ffmpeg.load()
    this.ready = true
  },

  computed: {
    videoSrc() {
      return URL.createObjectURL(this.video)
    }
  },

  methods: {
    // prettier-ignore
    async convertToGif() {
      ffmpeg.FS('writeFile', 'test.mp4', await fetchFile(this.video))
      await ffmpeg.run('-i', 'test.mp4', '-t', '2.5', '-ss', '2.0', '-f', 'gif', 'out.gif')
      const data = ffmpeg.FS('readFile', 'out.gif')
      const url = URL.createObjectURL(new Blob([data.buffer], { type: 'image/gif' }))
      this.gif = url
    }
  }
}
</script>
