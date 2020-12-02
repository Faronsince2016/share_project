<template>
  <page-header-wrapper
    title="质检记录"
    :back="back">
    <size-card :sizeData="sizeData"></size-card>
    <defect-card :defectData="defectData"></defect-card>
  </page-header-wrapper>
</template>

<script>
import DefectCard from './record/defect_card'
import SizeCard from './record/size-card'
import recordData from '@/views/dashboard/record/test_data'

export default {
  name: 'BatchDetail',
  components: { DefectCard, SizeCard },
  props: {
    deviceUid: {
      type: String,
      default: null,
      require: false
    },
    detectTime: {
      type: String,
      default: null,
      require: false
    }
  },
  data () {
    return {
      id: this.$route.params.recordID,
      recordData,
      loading: false,
      sizeData: null,
      defectData: null,
      sizeStandard: null
    }
  },
  created () {
    this.$nextTick(() => {
      this.loadData()
    })
  },
  methods: {
    back () {
      console.log('back')
    },
    async loadData () {
      this.defectData = this.recordData.defect_data
      this.sizeData = this.recordData.size_data
    }
  }
}
</script>

<style lang="less" scoped>

.item-status {
}

</style>
