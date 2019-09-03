/<template>
    <div class="q-pa-lg">

        <div class="row row-xl justify-center">
            <div class="col-3">
                <q-input
                    outlined
                    v-model.number="wireGap"
                    label="d="
                    style="max-width: 300px"
                    type="number"
                    debounce="500"
                    suffix="мм"
                    mask="#.##"
                    fill-mask="1.00"
                    reverse-fill-mask
                    dense
                    :decimals="2"
                    :step="0.1"
                    :rules="[ isValidWireGap ]"
                />
                <q-input
                    outlined
                    v-model.number="loopCount"
                    label="n="
                    style="max-width: 300px"
                    type="number"
                    debounce="500"
                    suffix="шт"
                    mask="#№"
                    fill-mask="2"
                    reverse-fill-mask
                    dense
                    :decimals="0"
                    :step="1"
                    :rules="[ isValidLoopCount ]"
                />
                <q-input type="file" @change="onFileUpload"/>
                <br/>
                <div class="row rox-xl">
                    <q-btn class="col-auto" color="primary" @click="onCalcClick" :disable="isReadyToCalc">Рассчитать</q-btn>
                </div>
            </div>
            <div class="col-auto">&nbsp;&nbsp;&nbsp;</div>
            <div class="col-6" style="border: 1px solid green">
                .col-12 .col-md-auto (Variable width content)
            </div>
        </div>

    </div>
</template>

<style>
</style>

<script>
export default {
  data: function () {
    return {
      wireGap: 0.5,
      loopCount: 2,
      isFileReady: false
    }
  },
  methods: {
    isValidWireGap (val) {
      return (val > 0) && (val < 10)
    },
    isValidLoopCount (val) {
      return (val > 0) && (val < 10)
    },
    onCalcClick () {
      if (!this.isValidWireGap(this.wireGap) || !this.isValidLoopCount(this.loopCount)) return
      console.log('click')
      this.wireGap = 5
    },
    onFileUpload (e) {
      const file = e.target.files[0]
      const reader = new FileReader()

      reader.onload = e => this.onFileLoaded(e.target.result)
      reader.readAsText(file)
    },
    onFileLoaded (contents) {
      console.log(contents)
    }
  },
  computed: {
    isReadyToCalc () {
      return !(this.isValidWireGap(this.wireGap) && this.isValidLoopCount(this.loopCount) && this.isFileReady)
    }
  },
  name: 'PageIndex'
}
</script>
