<template>
  <h2>The digital photocopier</h2>
  <p>Upload an image and get it back in photocopied form. Useful in situations where you have to put your signature on something, you don't want to waste paper, you have mspaint available, but the recipient of the signature insists that you send something that looks like it has been scanned.</p>
  <div class="row">
    <div class="col col-sm-12 col-lg-4">
      <h3>Configuration</h3>
      <form>
        <div class="form-group">
          <label for="file">Select an image</label>
          <input type="file" class="form-control-file" accept="image/*" @change="onChange" />
        </div>
        <div class="form-check">
          <input type="checkbox" class="form-check-input" id="grayscale" v-model="grayscale" />
          <label class="form-check-label" for="grayscale">Grayscale</label>
        </div>
        <div class="form-check">
          <input type="checkbox" class="form-check-input" id="blurcompose" v-model="blurcompose" />
          <label class="form-check-label" for="bluecompose">Blur and compose</label>
        </div>
        <div class="form-group">
          <input v-model="stretchPercentage" class="form-control" id="stretchPercentage" type="range" min="0" max="10" step="1" />
          <label for="stretchPercentage">Stretch: {{ stretchPercentage }}%</label>
        </div>
        <div class="form-group">
          <input v-model="rotationAngle" class="form-control" id="rotationAngle" type="range" min="-5" max="5" step="0.1" />
          <label for="rotationAngle">Rotation: {{ rotationAngle }}°</label>
        </div>
      <button type="button" class="btn btn-primary" @click="produceOutput">Generate</button>
    </form>
    </div>
    <div class="col col-sm-12 col-lg-8">
      <h3>Output</h3>
      <div v-if="generating">Generating; please wait.</div>
      <canvas id="photocopy" />
    </div>
  </div>
</template>

<script>
import {
  initializeImageMagick,
  ColorSpace,
  CompositeOperator,
  ImageMagick,
  Percentage
} from '@imagemagick/magick-wasm'

export default {
  name: 'Photocopy',
  props: {
    msg: String
  },
  data () {
    return {
      generating: false,
      blurcompose: true,
      grayscale: true,
      rotationAngle: 1.5,
      stretchPercentage: 3,
      image: null
    }
  },
  methods: {
    onChange (e) {
      initializeImageMagick().then(() => {
        const file = e.target.files[0]
        this.image = file
      })
    },
    async produceOutput () {
      if (this.image === null) return
      this.generating = true
      const arrayBuffer = await this.image.arrayBuffer()
      const array = new Uint8Array(arrayBuffer)
      ImageMagick.read(array, image => {
        if (this.grayscale) { image.colorSpace = ColorSpace.Gray }

        image.clone(clone => {
          if (this.blurcompose) {
            clone.blur(0, 1)
            image.composite(clone, CompositeOperator.DivideSrc)
          }
          if (this.stretchPercentage !== 0) {
            image.contrastStretch(new Percentage(this.stretchPercentage), new Percentage(0))
          }
          if (this.rotationAngle !== 0) { image.rotate(this.rotationAngle) }
          const canvas = document.getElementById('photocopy')
          image.writeToCanvas(canvas)
        })
      })
      this.generating = false
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
