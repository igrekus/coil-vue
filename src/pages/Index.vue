<template>
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
                <div class="row row-xl">
                    <input type="file" @change="onFileUpload"/>
                </div>
                <div class="row row-xl">
                    &nbsp;
                </div>
                <div class="row rox-xl justify-between">
                    <q-btn class="col-auto" color="primary" @click="onCalcClick" :disable="isCalcDisabled">Рассчитать</q-btn>
                    <q-btn class="col-auto" color="secondary" @click="onSaveGcodeClick">Сохранить CNC</q-btn>
                </div>
            </div>
            <div class="col-auto">&nbsp;&nbsp;&nbsp;</div>
            <div id="mainSvgDiv" class="col-6" style="border: 1px solid green">
                <svg id="mainSvgView">

                </svg>
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
      rawSvgString: '',

      snap: null,

      svg: null,
      mainSvgView: null,
      content: null,

      camPath: null,
      camPathSvg: null,

      combinedGeometry: []
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
      this.snap = require('snapsvg')

      this.mainSvgView = this.snap('#mainSvgView')

      this.svg = this.snap.parse(this.rawSvgString)

      // this.content = this.mainSvgView.group()
      // this.content.clear()
      // this.content.append(svg)

      this.mainSvgView.clear()
      this.mainSvgView.append(this.svg)

      this.updateSvgSize()

      this.generateToolPath()
    },
    onSaveGcodeClick () {
      console.log('save gcode')
    },
    onFileUpload (e) {
      const file = e.target.files[0]
      const reader = new FileReader()

      reader.onload = e => this.onFileLoaded(e.target.result)
      reader.readAsText(file)
    },
    onFileLoaded (contents) {
      this.rawSvgString = contents
    },

    // business logic
    generateToolPath () {
      // let startTime = Date.now()
      // let generatingToolpath = true

      // TODO extract removeToolPaths()
      if (this.camPathSvg) {
        this.camPathSvg.remove()
        this.camPathSvg = null
        this.camPath = null
      }

      let sourceGeometry = this.snap.path.toAbsolute(this.svg.paper.select('path'))
      if (!sourceGeometry) {
        console.log('wrong source geometry, should be a closed path')
      }

      let geometry = this.snap2clipper(sourceGeometry)

      let coil = this.coil(geometry, this.wireGap, this.loopCount)
      console.log('coil res', coil)

      // this.camPath = coil

      // var path = jscut.priv.path.getSnapPathFromClipperPaths(jscut.priv.cam.getClipperPathsFromCamPaths(self.camPath()), svgViewModel.pxPerInch())
      // if (path != null && path.length > 0) { self.toolPathSvg = toolPathsGroup.path(path).attr('class', 'camPath') }
      //
      // if (options.profile) { console.log('generateToolPath: ' + (Date.now() - startTime)) }
      //
      // self.enabled(true)
      // generatingToolpath = false
      // toolPathsChanged()
    },
    coil (geometry, wireGap, loopCount) {
      geometry = [geometry] // hack

      let current = this.offset(geometry, 0)
      let allPaths = []

      wireGap = 50
      loopCount = 2

      console.log('geometry', geometry)
      console.log('current start', current)
      for (let loops = 0; loops < loopCount + 1; ++loops) {
        allPaths = current.concat(allPaths)
        current = this.offset(current, -wireGap)
        console.log('current', loops, current)
      }

      // allPaths = allPaths.reverse()
      //
      // let res = []
      // for (let i = 0; i < allPaths.length - 1; ++i) {
      //   for (let j = 0; j < allPaths[i].length; ++j) {
      //     res.push(allPaths[i][j])
      //   }
      // }
      // let lastPath = allPaths[allPaths.length - 1]
      // res.push(lastPath[0])
      //
      // let output = [
      //   {
      //     path: res,
      //     safeToClose: false
      //   }
      // ]
      let output = allPaths

      return output
    },

    // helpers
    updateSvgSize () {
      let $ = window.$
      $('#mainSvgView').attr({
        width: $('#mainSvgDiv').width(),
        height: Math.max(10, $(window).height() - 120),
        preserveAspectRatio: 'xMinYMin meet'
      })
      // let bbox = this.mainSvgView.getBBox()
      // $('#mainSvgView').get(0).setAttribute('viewBox', (bbox.x - 2) + ' ' + (bbox.y - 2) + ' ' + (bbox.w + 4) + ' ' + (bbox.h + 4))
    },
    snap2clipper (path) {
      // const mmToClipperScale = 10000
      const pixPerInch = 96
      const inchToClipperScale = 100000
      const scale = inchToClipperScale / pixPerInch

      function getClipperPointFromSnapPoint (x, y) {
        return {
          // X: x * scale,
          // Y: y * scale
          X: Math.round(x * scale),
          Y: Math.round(y * scale)
        }
      }

      if (path.length < 2 || path[0].length !== 3 || path[0][0] !== 'M') {
        console.log('Error: path does not begin with M')
        return null
      }

      let currentPoint = getClipperPointFromSnapPoint(path[0][1], path[0][2])
      let newPoint = currentPoint

      let result = [currentPoint]

      for (let i = 1; i < path.length; ++i) {
        const subpath = path[i]
        switch (subpath[0]) { // line draw command
        case 'L':
          newPoint = getClipperPointFromSnapPoint(subpath[1], subpath[2])
          break
        case 'H':
          newPoint = getClipperPointFromSnapPoint(subpath[1], currentPoint.Y)
          break
        case 'V':
          newPoint = getClipperPointFromSnapPoint(currentPoint.X, subpath[1])
          break
        // case 'Z':
        // case 'z':
        //   newPoint = result[0]
        //   break
        }
        result.push(newPoint)
        currentPoint = newPoint
      }
      return result.slice(0, result.length - 1)
    },
    offset (paths, amount, joinType, endType) {
      const inchToClipperScale = 100000
      const arcTolerance = inchToClipperScale / 40000
      // const cleanPolyDist = inchToClipperScale / 100000

      if (typeof joinType === 'undefined') {
        joinType = window.clipper.JoinType.jtRound
      }
      if (typeof endType === 'undefined') {
        endType = window.clipper.EndType.etClosedPolygon
      }

      // bug workaround: join types are swapped in ClipperLib 6.1.3.2
      if (joinType === window.clipper.JoinType.jtSquare) {
        joinType = window.clipper.JoinType.jtMiter
      } else if (joinType === window.clipper.JoinType.jtMiter) {
        joinType = window.clipper.JoinType.jtSquare
      }

      let co = new window.clipper.ClipperOffset(2, arcTolerance)
      co.AddPaths(paths, joinType, endType)
      let offsetted = []
      co.Execute(offsetted, amount)
      return offsetted
    }
  },
  computed: {
    isCalcDisabled () {
      return (
        this.isValidWireGap(this.wireGap) &&
        this.isValidLoopCount(this.loopCount) &&
        (this.rawSvgString.length === 0)
      )
    }
  },
  name: 'PageIndex'
}
</script>
