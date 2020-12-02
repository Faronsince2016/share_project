<template>
  <div>
    <a-card title="外观缺陷检测结果" style="margin-top: 20px">
      <a-card-grid class="card-grid" style="width:15%">
        <div v-for="(item,i) in defectData" :key="i" class="img-item-box" @click="changeMainItem(item)">
          <a-card class="img-item">
            <img slot="cover" :alt="item.title" :src="item.img_src" />
            <a-card-meta :title="item.title" style="margin-top: 3px;margin-bottom: 3px">
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
      <a-card-grid class="card-grid" style="width:65%;border: 5px;margin: 0">
        <div id="canvas-box">
          <canvas id="label-canvas" style="display: block;width: 100%;height:100%"></canvas>
          <img id="main-img" :src="mainImg" style="max-width: 100%;display: none" alt="mainImg">
        </div>
      </a-card-grid>
      <a-card-grid class="card-grid" style="width:20%;min-height: 800px">
        <div class="sub-title">异常信息</div>
        <!--        <div class="err-tag" v-if="defectData" v-for="defect in item.defect_list" :key="defect.title">-->
        <!--          <a-badge :count="3" style="margin-right: 15px">-->
        <!--            <a-tag :color="defect.level | tagColorsFilter">-->
        <!--              <span class="tag-content">{{ defect.alias }}</span>-->
        <!--            </a-tag>-->
        <!--          </a-badge>-->
        <!--        </div>-->
      </a-card-grid>
    </a-card>
  </div>
</template>

<script>
import { fabric } from 'fabric'

const tagColors = {
  warning: {
    color: 'red',
    text: '异常'
  },
  error: {
    color: 'orange',
    text: '警告'
  }
}

export default {
  name: 'LabelMain',
  props: {
    id: {
      type: Number,
      default: null
    },
    defectData: {
      type: Array,
      default: null,
      require: true
    }
  },
  data () {
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
      flag: false
    }
  },
  watch: {
    // defectData() {
    //   console.log('this.defectData', this.defectData)
    // },
    async mainImg (newSrc) {
      // 重新渲染标注信息
      await new Promise((resolve, reject) => {
        this.fabricCanvas()
        // 绘制标记框
        this.item.defect_list.forEach(defectItem => {
            const pointList = defectItem.data
            const newPointList = [
              ['M'].concat(pointList[0]),
              ['L'].concat(pointList[1]),
              ['L'].concat(pointList[2]),
              ['L'].concat(pointList[3]),
              ['L'].concat(pointList[0]),
              ['z']
            ]
            const p = new fabric.Path(newPointList)
            p.selectable = false
            p.set({ fill: null, stroke: tagColors[defectItem.level].color, strokeWidth: 3 })
            this.fabricObj.add(p)
          }
        )
        this.fabricObj.renderAll()
      })
    }
  },
  computed: {},
  mounted () {
    this.$nextTick(() => {
      setTimeout(() => {
        this.loading = true
        this.fabricCanvas()
        this.fabricObjEvent()
        this.loading = false
      }, 500)
    })
  },
  methods: {
    /**
     * @desc 初始化fabric，添加图待标注图片到画布中。
     * */
    async fabricCanvas () {
      const canvasBox = document.getElementById('canvas-box')
      const labelCanvas = document.getElementById('label-canvas')
      labelCanvas.width = canvasBox.clientWidth || canvasBox.offsetWidth
      labelCanvas.height = canvasBox.clientHeight || canvasBox.offsetHeight
      this.fabricObj = new fabric.Canvas('label-canvas')
      this.fabricObj.selectable = false
      this.fabricObj.selection = false
      this.fabricObj.skipTargetFind = false

      const imgElement = document.getElementById('main-img')
      const imgWidth = imgElement.naturalWidth
      const imgHeight = imgElement.naturalHeight
      // 计算拉伸比例
      let scale
      if (imgWidth / imgHeight > labelCanvas.width / labelCanvas.height) {
        scale = labelCanvas.width / imgWidth
      } else {
        scale = labelCanvas.height / imgHeight
      }
      const img = new fabric.Image(imgElement, {
        scaleX: scale,
        scaleY: scale,
        zIndex: 2,
        selectable: false
      })
      this.fabricObj.add(img)
      this.fabricObj.renderAll()
    },
    /**
     * @desc事件监听
     * */
    fabricObjEvent () {
      // 开启标注模式
      // this.fabricObj.isDrawingMode = true
      this.fabricObj.on({
        'mouse:down': (e) => {
          this.doDrawing = true
          // 鼠标按下
          this.mouseFrom.x = e.pointer.x
          this.mouseFrom.y = e.pointer.y
        },
        'mouse:move': (e) => {
          // 鼠标移动
          if (this.moveCount % 2 && !this.doDrawing) {
            // 减少绘制频率
            return
          }
          this.moveCount++
          this.mouseTo.x = e.pointer.x
          this.mouseTo.y = e.pointer.y
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
            borderDashArray: [3, 3]
          })
        }

      })
    },
    drawing () {
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
        ['z']
      ]
      const fabricNew = new fabric.Path(pathArray, {
        left: this.mouseFrom.x,
        top: this.mouseFrom.y,
        stroke: '#ee0000',
        strokeWidth: 2,
        fill: 'rgba(255, 255, 255, 0)'
      })
      if (fabricNew) {
        this.fabricObj.add(fabricNew)
        this.fabricObj.selectable = false
        this.drawingObject = fabricNew
      }
    },
    async changeMainItem (item) {
      this.flag = true
      this.item = item
      this.mainImg = item.img_src
      if (this.fabricObj) this.fabricObj.clear().renderAll()
    }
  },
  filters: {
    tagColorsFilter (type) {
      return tagColors[type].color
    }
  }
}
</script>

<style lang="less" scoped>

.card-grid {
  height: 800px;
  overflow: scroll
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
  margin: 10px
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
