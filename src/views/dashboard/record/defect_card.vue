<template>
  <div>
    <a-card title="外观缺陷检测结果" style="margin-top: 20px">
      <a-card-grid class="card-grid" style="width: 15%">
        <div v-for="(item, i) in defectData" :key="i" class="img-item-box" @click="changeMainItem(item)">
          <a-card class="img-item">
            <img slot="cover" :alt="item.title" :src="item.img_src" />
            <a-card-meta :title="item.title" style="margin-top: 3px; margin-bottom: 3px">
              <template slot="description" style="margin-bottom: 3px">
                <div :key="defect.name" v-for="defect in item.defect_list" style="display: inline">
                  <a-tag class="micro-err-tag" :color="defect.level | tagColorsFilter" style="margin-bottom: 10px">
                    {{ defect.alias }}
                  </a-tag>
                </div>
              </template>
            </a-card-meta>
          </a-card>
        </div>
      </a-card-grid>
      <a-card-grid class="card-grid" style="width: 65%; border: 5px; margin: 0">
        <div id="canvas-box" ref="canvasBox">
          <canvas id="label-canvas" ref="c"></canvas>
        </div>
      </a-card-grid>
      <a-card-grid class="card-grid" v-if="selectItem" style="width: 20%; min-height: 800px">
        <div class="sub-title">异常信息</div>
        <div class="err-tag" v-for="(item, index) in selectItem.defect_list" :key="index">
          <a-badge :count="3" style="margin-right: 15px">
            <a-tag :color="item.level | tagColorsFilter">
              <span class="tag-content">{{ item.alias }}</span>
            </a-tag>
          </a-badge>
        </div>
      </a-card-grid>
    </a-card>
  </div>
</template>

<script>
import { fabric } from 'fabric'

const tagColors = {
  warning: {
    color: 'red',
    text: '异常',
  },
  error: {
    color: 'orange',
    text: '警告',
  },
}

export default {
  name: 'LabelMain',
  props: {
    id: {
      type: Number,
      default: null,
    },
    defectData: {
      type: Array,
      default: null,
      require: true,
    },
  },
  data() {
    return {
      mainImg: '',
      loading: true,
      fabricObj: null, // canvas 对象
      drawingObject: null, // 绘制对象
      moveCount: 1,
      mouseFrom: {},
      mouseTo: {},
      doDrawing: false,
      labelResult: {},
      flag: false,
      selectItem: null,
    }
  },
  watch: {
    defectData(data) {
      if (!data || !data[0]) return
      this.changeMainItem(data[0])
    },
    async mainImg(newSrc) {
      // 重新渲染标注信息
      await this.fabricCanvas()
    },
  },
  computed: {},
  mounted() {
    this.loading = true
    this.init(this.$refs.c, { enableRetinaScaling: true })
    this.fabricObjEvent()
    this.loading = false
  },
  methods: {
    getInner() {
      const { clientHeight: height, clientWidth: width } = this.$refs.canvasBox
      this.inner = { height, width }
    },
    init(c, option = {}) {
      this.fabricObj = new fabric.Canvas(
        c,
        Object.assign(
          {
            imageSmoothingEnabled: false,
            includeDefaultValues: false,
            selection: false,
            selectable: false,
            enableRetinaScaling: false,
            skipTargetFind: false,
          },
          option
        )
      )
    },
    setZoom(cover) {
      this.getInner()
      const img = this.fabricObj.item(0)
      const size = img || this.inner
      let zoom = 1
      const { height: eleHeight, width: eleWidth } = this.inner
      const rate = eleWidth / eleHeight
      const imgRate = size.width / size.height
      const ratio = cover ? imgRate < rate : imgRate > rate
      if (ratio) {
        zoom = eleWidth / size.width
      } else {
        zoom = eleHeight / size.height
      }
      this.fabricObj.setZoom(zoom)
      this.fabricObj.setWidth(eleWidth)
      this.fabricObj.setHeight(eleHeight)
      this.fabricObj.absolutePan({
        x: (size.width * zoom - eleWidth) * 0.5 + (isNaN(size.left * zoom) ? 0 : size.left * zoom),
        y: (size.height * zoom - eleHeight) * 0.5 + (isNaN(size.top * zoom) ? 0 : size.top * zoom),
      })
      this.fitZoom = zoom
      this.fabricObj.requestRenderAll()
    },
    drawPath() {
      // 绘制标记框
      this.selectItem.defect_list.forEach((defectItem) => {
        const pointList = defectItem.data
        const newPointList = [
          ['M'].concat(pointList[0]),
          ['L'].concat(pointList[1]),
          ['L'].concat(pointList[2]),
          ['L'].concat(pointList[3]),
          ['L'].concat(pointList[0]),
          ['z'],
        ]
        const p = new fabric.Path(newPointList)
        p.selectable = false
        p.set({ fill: null, stroke: tagColors[defectItem.level].color, strokeWidth: 3 })
        this.fabricObj.add(p)
      })
      this.fabricObj.renderAll()
    },
    clearImgElement() {
      if (this.imgElement) {
        this.imgElement.onload = null
        this.imgElement.onerror = null
        this.imgElement = null
      }
    },
    async clear() {
      this.fabricObj.clear()
      this.setZoom()
    },
    /**
     * @desc 初始化fabric，添加图待标注图片到画布中。
     * */
    async fabricCanvas() {
      this.clearImgElement()
      if (this.resolve) this.resolve()
      if (!this.mainImg) return
      await new Promise((resolve, reject) => {
        this.resolve = resolve
        this.imgElement = document.createElement('img')
        this.imgElement.onload = () => {
          this.clear()
          const namedImg = new fabric.Image(this.imgElement, {
            objectCaching: false,
            evented: false,
            selectable: false,
          })
          this.fabricObj.add(namedImg)
          this.drawPath()
          this.setZoom()
          this.clearImgElement()
          this.resolve = null
          resolve()
        }
        this.imgElement.onerror = (err) => {
          this.clearImgElement()
          this.resolve = null
          reject(err)
        }
        this.imgElement.src = this.mainImg
      })
    },
    /**
     * @desc事件监听
     * */
    fabricObjEvent() {
      // 开启标注模式
      // this.fabricObj.isDrawingMode = true
      this.fabricObj.on({
        'mouse:down': (e) => {
          this.doDrawing = true
          // 鼠标按下
          const { x, y } = this.fabricObj.getPointer(e.e, false)
          this.mouseFrom = new fabric.Point(x, y)
        },
        'mouse:move': (e) => {
          // 鼠标移动
          if (this.moveCount % 2 && !this.doDrawing) {
            // 减少绘制频率
            return
          }
          this.moveCount++
          const { x, y } = this.fabricObj.getPointer(e.e, false)
          this.mouseTo = new fabric.Point(x, y)
          this.drawing()
        },
        'mouse:up': (e) => {
          // 鼠标抬起
          this.moveCount = 1
          this.mouseTo.x = e.pointer.x
          this.mouseTo.y = e.pointer.y
          this.doDrawing = false
        },
        'selection:created': (e) => {
          e.target.set({
            transparentCorners: false,
            cornerColor: '#ff7a55',
            cornerStrokeColor: '#ff7a55',
            borderColor: 'red',
            cornerSize: 12,
            padding: 10,
            cornerStyle: 'circle',
            borderDashArray: [3, 3],
          })
        },
      })
    },
    drawing() {
      if (this.drawingObject) {
        this.fabricObj.remove(this.drawingObject)
      }
      // 使用path绘制矩形
      const pathArray = [
        ['M', this.mouseFrom.x, this.mouseFrom.y],
        ['L', this.mouseTo.x, this.mouseFrom.y],
        ['L', this.mouseTo.x, this.mouseTo.y],
        ['L', this.mouseFrom.x, this.mouseTo.y],
        ['L', this.mouseFrom.x, this.mouseFrom.y],
        ['z'],
      ]
      const fabricNew = new fabric.Path(pathArray, {
        left: this.mouseFrom.x,
        top: this.mouseFrom.y,
        stroke: '#ee0000',
        strokeWidth: 2,
        fill: 'rgba(255, 255, 255, 0)',
      })
      if (fabricNew) {
        this.fabricObj.add(fabricNew)
        this.fabricObj.selectable = false
        this.drawingObject = fabricNew
      }
    },
    async changeMainItem(item) {
      if (this.selectItem === item) return
      this.flag = true
      this.selectItem = item
      this.mainImg = item.img_src
      if (this.fabricObj) this.fabricObj.clear().renderAll()
    },
  },
  filters: {
    tagColorsFilter(type) {
      return tagColors[type].color
    },
  },
}
</script>

<style lang="less" scoped>
.card-grid {
  height: 800px;
  overflow: scroll;
}

.sub-title {
  height: 60px;
  line-height: 60px;
  color: #000000;
  font-size: 16px;
  font-weight: 500;
}

.err-tag {
  height: 20px;
  margin: 20px;
}

.tag-content {
  font-size: 20px;
  vertical-align: middle;
  margin: 10px;
}

.img-item-box {
  border: 1px solid #ffffff;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.img-item {
  max-height: 300px;
  max-width: 300px;
  margin-top: 10px;
  margin-bottom: 10px;
  text-align: center;
  vertical-align: middle;
}

.img-item:hover {
  box-shadow: 0 0 5px 5px #ccc;
}

#canvas-box {
  border: 1px solid #dddeea;
  min-height: 500px;
  position: relative;
  height: 100%;
  width: 100%;
  overflow: hidden;
}
</style>
