<template>
  <div ref="gantt"></div>
</template>

<script>
import { gantt } from 'dhtmlx-gantt'
import 'dhtmlx-gantt/codebase/locale/locale_cn'

export default {
  name: 'gantt',
  props: {
    tasks: {
      type: Object,
      default () {
        return { data: [], links: [] }
      }
    }
  },
  mounted: function () {
    // 定义日期格式，该日期格式用于解析数据集中的数据并将数据发送到服务器
    gantt.config.xml_date = '%Y-%m-%d'
    // 设置时间刻度的格式（X轴）
    gantt.config.date_scale = '%m月%d日'
    this._initGanttEvents()
    // 初始化dhtmlxGantt
    gantt.init(this.$refs.gantt)
    // loads data from a client-side resource
    gantt.parse(this.$props.tasks)
    this._initDataProcessor()
  },

  methods: {
    /**
     * 响应 dhtmlxGantt 内部事件
     * @private
     */
    _initGanttEvents: function () {
      if (!gantt.$_eventsInitialized) {
        gantt.attachEvent('onTaskSelected', (id) => {
          const task = gantt.getTask(id)
          this.$emit('task-selected', task)
        })

        gantt.attachEvent('onTaskIdChange', (id, newId) => {
          if (gantt.getSelectedId() === newId) {
            const task = gantt.getTask(newId)
            this.$emit('task-selected', task)
          }
        })

        gantt.$_eventsInitialized = true
      }
    },

    _initDataProcessor: function () {
      if (!gantt.$_dataProcessorInitialized) {
        // 创建新的dataProcessor实例并将其附加到甘特图
        // entity - "task"|"link"
        // action - "create"|"update"|"delete"
        // data - an object with task or link data
        // id – the id of a processed object (task or link)
        gantt.createDataProcessor((entity, action, data, id) => {
          this.$emit(`${entity}-updated`, id, action, data)
        })

        gantt.$_dataProcessorInitialized = true
      }
    }
  }
}
</script>

<style>
  @import "~dhtmlx-gantt/codebase/dhtmlxgantt.css";
</style>
