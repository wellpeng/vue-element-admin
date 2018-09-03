<template>
  <div id="chartVue" class="dashboard-editor-container">
    <power-drag
      ref="cyGridster"
      :your-list="myList"
      :base-margin-left="baseMarginLeft"
      :base-margin-top="baseMarginTop"
      :base-width="baseWidth"
      :base-height="baseHeight"
      :drag-start="dragStart"
      :dragging="dragging"
      :drag-end="dragEnd"
      :resize-start="resizeStart"
      :resizing="resizing"
      :resize-end="resizeEnd">
      <template v-for="(item,index) in myList" :slot="'slot' + index">
        <div :key="item.id" class="chart-wrapper dragHandle">
          <div class="tool">
            <span @click="deleteItem(index)">X</span>
          </div>
          <div :key="item.id" class="chart">
            <component :key="item.id" :is="item.chart" :ref="'chart' + index" />
          </div>
        </div>
      </template>
    </power-drag>
  </div>
</template>
<script>
import drag from 'vue-power-drag'
import mock from 'mockjs'
import $ from 'jquery'
import RaddarChart from './components/RaddarChart'
import PieChart from './components/PieChart'
import BarChart from './components/BarChart'
import LineChart from './components/LineChart'
import _ from 'lodash'

const lineChartData = {
  newVisitis: {
    expectedData: [100, 120, 161, 134, 105, 160, 165],
    actualData: [120, 82, 91, 154, 162, 140, 145]
  },
  messages: {
    expectedData: [200, 192, 120, 144, 160, 130, 140],
    actualData: [180, 160, 151, 106, 145, 150, 130]
  },
  purchases: {
    expectedData: [80, 100, 121, 104, 105, 90, 100],
    actualData: [120, 90, 100, 138, 142, 130, 130]
  },
  shoppings: {
    expectedData: [130, 140, 141, 142, 145, 150, 160],
    actualData: [120, 82, 91, 154, 162, 140, 130]
  }
}

const chartInstanceBox = {}

export default {
  name: 'App',
  components: {
    'power-drag': drag,
    RaddarChart,
    PieChart,
    BarChart,
    LineChart
  },
  data() {
    const list = mock.mock({
      myList: [{
        'id': 1,
        'x': 1,
        'y': 20,
        'sizex': 10,
        'sizey': 10,
        'chart': 'PieChart'
      }, {
        'id': 2,
        'x': 1,
        'y': 1,
        'sizex': 10,
        'sizey': 10,
        'chart': 'BarChart'
      }, {
        'id': 4,
        'x': 11,
        'y': 1,
        'sizex': 10,
        'sizey': 10,
        'chart': 'RaddarChart'
      }, {
        'id': 6,
        'x': 11,
        'y': 20,
        'sizex': 10,
        'sizey': 10,
        'chart': 'PieChart'
      }, {
        'id': 8,
        'x': 21,
        'y': 1,
        'sizex': 10,
        'sizey': 10,
        'chart': 'BarChart'
      }, {
        'id': 10,
        'x': 22,
        'y': 20,
        'sizex': 10,
        'sizey': 10,
        'chart': 'PieChart'
      }, {
        'id': 3,
        'x': 31,
        'y': 20,
        'sizex': 10,
        'sizey': 10,
        'chart': 'PieChart'
      }, {
        'id': 5,
        'x': 21,
        'y': 20,
        'sizex': 10,
        'sizey': 10,
        'chart': 'PieChart'
      }, {
        'id': 7,
        'x': 33,
        'y': 20,
        'sizex': 6,
        'sizey': 6,
        'chart': 'PieChart'
      }, {
        'id': 9,
        'x': 33,
        'y': 50,
        'sizex': 6,
        'sizey': 6,
        'chart': 'PieChart'
      }]
    })
    return {
      lineChartData: lineChartData.newVisitis,
      myList: list.myList,
      baseWidth: 0,
      baseHeight: 0
    }
  },
  created() {
    // 屏幕适配，使得当前布局能在所有分辨率下适用，示例是在1366*638分辨率下完成
    const screenWidth = window.innerWidth
    const screenHeight = window.innerHeight
    this.baseWidth = 10 // * (screenWidth / 900)
    this.baseHeight = 10 // * (screenHeight / 638)
    this.baseMarginLeft = 20 * (screenWidth / 1200)
    this.baseMarginTop = 20 * (screenHeight / 638)
    this.$nextTick(function() {
      $('.dragAndResize').css('width', 'calc(100% - ' + (this.baseMarginLeft) + 'px)')
    })
  },
  mounted() {
    const gridster = this.$refs['cyGridster'] // 获取 gridster 实例
    const vm = this
    gridster.afterInitOk(function() {
      _.forEach(vm.myList, function(item, index) {
        const chartNode = vm.$refs['chart' + index][0]
        // console.info(chartNode)
        chartNode.chart.resize()
        chartInstanceBox['chart' + index] = chartNode
      })
    })
    gridster.init() // 在适当的时候初始化布局组件
  },
  methods: {
    getList() {
      const gridster = this.$refs['cyGridster'] // 获取 gridster 实例
      console.log(JSON.stringify(gridster.getList()))
    },
    deleteItem(index) {
      const gridster = this.$refs['cyGridster'] // 获取 gridster 实例
      gridster.removeItem(index) // 此时会在 this.myList的index位置将item置为{}，目的是为了不让vue重新渲染整个v-for。
      // 注意，这里删除布局框并不会删除里面的组件，组件需要自己用v-if来控制销毁，如果是highchart，必须手动调用chartInstance.$destroy()
    },
    dragStart(e, item, index) {},
    dragging(e, item, index) {},
    dragEnd(e, item, index) {},
    resizeStart(e, item, index) {},
    resizing(e, item, index) {},
    resizeEnd(e, item, index) {
      chartInstanceBox['chart' + index].chart.resize()
    },
    handleSetLineChartData(type) {
      this.lineChartData = lineChartData[type]
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .dashboard-editor-container {
    padding: 32px;
    background-color: rgb(240, 242, 245);
    .chart-wrapper {
      background: #fff;
      padding: 16px 16px 0;
      margin-bottom: 32px;
    }
  }
  #chartVue {
    width: 100%;
    padding: 1.5em 0 1.5em 0;
    .head {
      height: 50px;
      border-bottom: 1px dashed;
      b {
        cursor: pointer;
        font-weight: bold;
        margin-left: 1rem;
      }
      .arrow {
        font-size: 20px;
        position: relative;
        top: 2px;
        margin-right:10px;
      }
    } // 拖动布局容器样式
    .dragAndResize {
      // 布局框样式
      .item {}
      .dragHandle {
        // 拖动手柄样式
        padding: 1.6rem 0.5rem 0.5rem 0.2rem !important;
        height: 100%; // 自定义内容样式
        .tool {
          position: absolute;
          right: .5rem;
          top: .5rem;
          cursor: pointer;
          font-weight: bold;
        }
        .chart {
          width: 100%;
          height: 100%;
          overflow: hidden;
          cursor: default;
        }
      }
    }
  }
</style>
