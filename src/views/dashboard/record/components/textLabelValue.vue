<template>
  <div class="text-label-value" style="line-height: 30px">
    <span class="label">
      <span style="width: 50px;overflow: hidden;text-overflow: ellipsis;">
        {{ item.alias || item.name }}
      </span>
    </span>
    <div class="standard-wrap">
      <div class="center-box">
        <count-to
          ref="countTo"
          v-if="item.standard_value !== undefined"
          :start-val="standard.start"
          :end-val="standard.end"
          :duration="300"
          :decimals="2"
        />
        <div class="error-value plus">
          <count-to
            ref="countTo"
            prefix="+"
            v-if="item.es !== undefined"
            :start-val="error_plus.start"
            :end-val="error_plus.end"
            :duration="300"
            :decimals="2"
          />
        </div>
        <div class="error-value minus">
          <count-to
            ref="countTo"
            prefix="-"
            v-if="item.ei !== undefined"
            :start-val="error_minus.start"
            :end-val="error_minus.end"
            :duration="300"
            :decimals="2"
          />
        </div>
      </div>
    </div>
    <div class="measured-wrap">
      <div class="center-box" :class="{ visible: !measuredHide }">
        <count-to
          ref="countTo"
          v-if="item.measure_value !== undefined"
          :start-val="measured.start"
          :end-val="measured.end"
          :duration="300"
          :decimals="2"
        />
        <span class="over-value" v-if="item.over_value && item.over_value > 0">
          {{ item.over_value }}
          <a-icon class="arrow" type="arrow-up" />
        </span>
        <span class="over-value" v-else-if="item.over_value">
          {{ -item.over_value }}
          <a-icon type="arrow-down" class="arrow" />
        </span>
      </div>
    </div>
    <div class="status el-input">
      <div class="status-wrap" :class="statusClass">
        <!--        <i v-if="item.correct">*</i>-->
        <span>{{ statusClass.ok ? 'OK' : 'NG' }}</span>
      </div>
    </div>
  </div>
</template>
<script type="text/babel">
import _ from 'lodash'
import countTo from 'vue-count-to'
import * as utils from '@/utils/util'

export default {
  name: 'TextLabelValue',
  data () {
    return {
      measured: {
        start: 0,
        end: 0
      },
      standard: {
        start: 0,
        end: 0
      },
      error_plus: {
        start: 0,
        end: 0
      },
      error_minus: {
        start: 0,
        end: 0
      },
      measuredHide: false
    }
  },
  components: { countTo },
  props: {
    // eslint-disable-next-line vue/require-prop-types
    item: {
      type: Object,
      default () {
        return {}
      }
    },
    // eslint-disable-next-line vue/require-prop-types
    cache: {},
    // eslint-disable-next-line vue/require-prop-types
    index: {}
  },
  computed: {
    canEdit () {
      return (
        this.cache &&
        !this.cache.saved &&
        !['diff', 'summation'].includes(this.item.type) &&
        !this.cache.loading &&
        this.index === this.cache.index
      )
    },
    getLabelActive () {
      return _.get(this.cache, 'act_obj.item.uid') === this.item.uid
    },
    statusClass () {
      const status = !utils.isRealNum(this.item.over_value) || this.item.over_value === 0
      return {
        ok: status,
        ng: !status,
        'is-disabled': !this.canEdit && !status
      }
    }
  },
  methods: {},
  watch: {
    item: {
      handler: function (item) {
        this.measured.start = Number.isNaN(this.measured.end) ? 0 : this.measured.end
        this.measured.end = item.measure_value - 0
        this.standard.start = Number.isNaN(this.standard.end) ? 0 : this.standard.end
        this.standard.end = item.standard_value
        this.error_plus.start = Number.isNaN(this.error_plus.end) ? 0 : this.error_plus.end
        this.error_plus.end = item.es
        this.error_minus.start = Number.isNaN(this.error_minus.end) ? 0 : this.error_minus.end
        this.error_minus.end = item.ei
      },
      immediate: true
    }
  }
}
</script>
<style lang="less" scoped>
.text-label-value {
  height: 100%;
  width: 100%;
  display: flex;
  align-items: center;
  font-size: 16px;
  border: 1px solid #000;

  > * {
    flex: 1;
    padding: 9px 5px;
    text-align: center;
    line-height: 21px;
    border-right: 1px solid #919191;
    height: 58px;
    display: flex;
    align-items: center;
    justify-content: center;

    &:last-child {
      border: none;
    }
  }

  .label {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    width: 50px
  }

  .visibility {
    visibility: hidden;
  }

  .over-value {
    color: #d22a42;
    font-weight: bold;
    font-size: 15px;
    position: relative;
    vertical-align: bottom;

    .arrow {
      position: absolute;
      right: -15px;
      top: 0;
    }
  }

  .measured-wrap {
    position: relative;
    padding-right: 10px;

    .center-box {
      padding-right: 0;
      min-width: 100px;
      text-align: center;
      margin: auto;
      transition: color 0.2s;
      opacity: 0;

      &.visible {
        opacity: 1;
      }
    }
  }

  .center-box {
    margin: 0 auto;
    display: table;
    padding-right: 35px;
    position: relative;
    text-align: left;

    .error-value {
      font-size: 12px;
      position: absolute;
      height: 17px;
      line-height: 17px;
      right: 0;

      &.minus {
        bottom: -8px;
      }

      &.plus {
        top: -9px;
      }
    }
  }

  .status {
    .status-wrap {
      position: relative;
      font-weight: bold;
      text-align: center;
      color: #fff;
      height: 100%;
      line-height: 40px;
      border-radius: 2px;
      margin: 0 6px;
      transition: all 0.1s;
      width: 100%;

      > i {
        position: absolute;
        left: 5px;
        top: -10px;
      }

      &.ok {
        background: #00ba80;
      }

      &.ng {
        background: #d22a42;
        cursor: pointer;
      }

      &.is-disabled {
        cursor: not-allowed;
      }
    }
  }
}
</style>
