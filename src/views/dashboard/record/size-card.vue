<template>
  <div>
    <a-card title="尺寸测量结果" :hoverable="true" style="margin-top: 24px;width: auto">
      <a-card-grid class="card-grid" style="width:60%">
        <FeatureImg :src="sizeImg"></FeatureImg>
      </a-card-grid>
      <a-card-grid class="card-grid" style="width:40%" :hoverable="true">
        <div class="text-board">
          <div class="name-wrap">
            <div class="name">测量项</div>
            <div class="name">标准值</div>
            <div class="name">测量值</div>
            <div class="name">结果</div>
          </div>
          <div class="text-list">
            <text-label-value v-for="(item, i) in sizeData" :key="i" :item="item"></text-label-value>
          </div>
        </div>
      </a-card-grid>
    </a-card>
  </div>
</template>

<script>
import TextLabelValue from './components/textLabelValue'
import FeatureImg from './components/feature-img'

const statusMap = {
  NG: {
    status: '#ae0000',
    text: '异常'
  },
  OK: {
    status: '#097e00',
    text: '正常'
  }
}

const sizeImg = 'https://bzserve.luosi.com/img/spec/4703/d1/x0/p0/l0/m0.png?s=e783afc8bcf5a034579fa8f58b5d1a7a0416879a&n=sRjDjQY5&t=1604320047996'

export default {
  name: 'SizeCard',
  components: { TextLabelValue, FeatureImg },
  props: {
    sizeData: {
      type: Array,
      default: null,
      require: true
    },
    drawings: {
      type: String,
      default: '',
      require: true
    }
  },
  data () {
    return {
      sizeImg,
      loading: false
    }
  },
  methods: {},
  filters: {
    statusFilter (type) {
      return statusMap[type].text
    },
    statusTypeFilter (type) {
      return statusMap[type].status
    }
  }
}
</script>

<style lang="less" scoped>
.card-grid {
  height: 800px;
  overflow: hidden
}

.size-img {
  width: auto;
  height: auto;
  max-width: 1080px;
  max-height: 800px;
  margin: auto;
}

.text-board {
  position: relative;
  height: 100%;
  width: 100%;
  display: flex;
  flex-direction: column;
  text-align: center;
  vertical-align: middle;

  .text-label-value {
    height: auto;

    > * {
      height: auto;
    }
  }

  .text-list {
    display: flex;
    flex-direction: column;
    overflow: auto;
    overflow-x: hidden;
    height: calc(100% - 26px);

    .board-step-wrap {
      transition: all 0.15s;
      border: 2px solid #151515;

      .step-title {
        height: 28px;
        line-height: 24px;
        font-size: 14px;
        text-align: left;
        padding-left: 10px;
        color: #fff;
        background: #151515;
      }

      &.ok {
        .step-title {
          background: #00ba80;
        }
      }

      &.ng {
        .step-title {
          background: #d22a42;
        }
      }
    }
  }

  .unit {
    font-size: 12px;
  }

  .name-wrap {
    display: flex;
    align-items: center;
    padding: 15px 0;
    flex-shrink: 0;
    border: 1px solid #000;

    .name {
      text-align: center;
      padding: 0 5px;
      font-size: 20px;
      font-weight: bold;
      vertical-align: middle;
    }

    > * {
      flex: 1;
    }
  }
}
</style>
