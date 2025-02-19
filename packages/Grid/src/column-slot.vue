<template>
  <el-table-column
    v-if="getColumnProp(column, 'hide')"
    :key="column.prop"
    :prop="column.prop"
    :label="column.label"
    filter-placement="bottom-end"
    :filters="getColumnProp(column, 'filters')"
    :filter-method="getColumnProp(column, 'filterMethod') ? handleFiltersMethod : undefined"
    :filter-multiple="vaildData(column.filterMultiple, true)"
    :show-overflow-tooltip="column.overHidden"
    :min-width="column.minWidth"
    :sortable="getColumnProp(column, 'sortable')"
    :render-header="column.renderHeader"
    :align="column.align || grid.tableOption.align"
    :header-align="column.headerAlign || grid.tableOption.headerAlign"
    :width="getColumnProp(column, 'width')"
    :fixed="getColumnProp(column, 'fixed')"
  >
    <template slot="header" slot-scope="{ $index }">
      <slot
        v-if="grid.getSlotName(column, 'H', grid.$scopedSlots)"
        :name="grid.getSlotName(column, 'H')"
        v-bind="{ column, $index }"
      ></slot>
      <el-popover
        v-else
        placement="bottom"
        :disabled="(grid.objectOption[column.prop] || {}).screen !== true"
        trigger="hover"
      >
        <el-input
          v-model="(grid.objectOption[column.prop] || {}).screenValue"
          type="text"
          :placeholder="`请输入 ${column.label} 筛选关键字`"
          size="mini"
        ></el-input>
        <span slot="reference">{{ column.label }}</span>
      </el-popover>
    </template>
    <template slot-scope="{ row, $index }">
      <el-form-item
        v-if="row.$cellEdit && column.cell"
        :prop="grid.isTree ? '' : `list.${$index}.${column.prop}`"
        :label="vaildLabel(column, row, ' ')"
        :label-width="vaildLabel(column, row, '1px')"
        :rules="column.rules"
      >
        <el-tooltip
          :content="(grid.listError[`list.${$index}.${column.prop}`] || {}).msg"
          :disabled="!(grid.listError[`list.${$index}.${column.prop}`] || {}).valid"
          placement="top"
        >
          <slot
            v-if="grid.getSlotName(column, 'F', grid.$scopedSlots)"
            v-bind="{
              row: row,
              dic: grid.DIC[column.prop],
              size: grid.isMediumSize,
              index: $index,
              disabled: grid.btnDisabledList[$index],
              label: handleShowLabel(row, column, grid.DIC[column.prop]),
              $cell: row.$cellEdit,
            }"
            :name="grid.getSlotName(column, 'F')"
          ></slot>
          <form-temp
            v-else
            v-model="row[column.prop]"
            :column="column"
            :size="grid.isMediumSize"
            :dic="(grid.cascaderDIC[$index] || {})[column.prop] || grid.DIC[column.prop]"
            :props="column.props || grid.tableOption.props"
            :readonly="column.readonly"
            :disabled="grid.disabled || grid.tableOption.disabled || column.disabled || grid.btnDisabledList[$index]"
            :clearable="vaildData(column.clearable, false)"
            v-bind="$uploadFun(column, grid)"
            :column-slot="grid.mainSlot"
            @change="columnChange($index, row, column)"
          >
            <template v-for="item in grid.mainSlot" :slot="item" slot-scope="scope">
              <slot v-bind="scope" :name="item"></slot>
            </template>
          </form-temp>
        </el-tooltip>
      </el-form-item>
      <slot
        v-else-if="grid.$scopedSlots[column.prop]"
        :row="row"
        :index="$index"
        :dic="grid.DIC[column.prop]"
        :size="grid.isMediumSize"
        :label="handleShowLabel(row, column, grid.DIC[column.prop])"
        :name="column.prop"
      ></slot>
      <template v-else>
        <span v-if="['img', 'upload'].includes(column.type)">
          <div class="avue-grid__img">
            <img
              v-for="(item, index) in getImgList(row, column)"
              :key="index"
              :src="item"
              v-bind="allParams(item)"
              @click.stop="openImg(getImgList(row, column), index)"
            />
          </div>
        </span>
        <span v-else-if="'url' === column.type">
          <el-link
            v-for="(item, index) in corArray(row[column.prop], column.separator)"
            :key="index"
            type="primary"
            :href="item"
            :target="column.target || '_blank'"
            >{{ item }}</el-link
          >
        </span>
        <span v-else-if="'rate' === column.type">
          <avue-rate v-model="row[column.prop]" disabled />
        </span>
        <i
          v-else-if="'color' === column.type"
          class="avue-grid__color"
          :style="{ backgroundColor: row[column.prop] }"
        ></i>
        <i v-else-if="'icon' === column.type" class="avue-grid__icon" :class="row[column.prop]"></i>
        <span v-else-if="column.html" v-html="handleDetail(row, column)"></span>
        <span v-else v-text="handleDetail(row, column)"></span>
      </template>
    </template>
  </el-table-column>
</template>

<script>
let count = {}
import { detail } from './core/detail'
import { DIC_PROPS, DIC_SPLIT } from './global/variable'
import { sendDic } from './core/dic'
import formTemp from './core/components/form/index'
import { vaildData, setPx } from './utils/util'
import { validatenull } from './utils/validate'

export default {
  name: 'ColumnSlot',
  inject: ['dynamic', 'grid'],
  components: {
    formTemp,
  },
  props: {
    column: Object,
    columnOption: Array,
  },
  created() {
    const list = ['getColumnProp', 'handleFiltersMethod', 'handleFilters']
    Object.keys(this.dynamic).forEach((ele) => {
      if (list.includes(ele)) {
        this[ele] = this.dynamic[ele]
      }
    })
  },
  methods: {
    vaildData,
    allParams(item) {
      return {
        is: this.$typeList.video.test(item) ? 'video' : 'img',
      }
    },
    vaildLabel(column, row, val) {
      if (column.rules && row.$cellEdit) {
        return val
      }
    },
    handleShowLabel(row, column, DIC) {
      let result = ''
      if (!validatenull(DIC)) {
        result = detail(row, column, this.grid.tableOption, DIC)
        row['$' + column.prop] = result
      }
      return result
    },
    columnChange($index, row, column) {
      if (validatenull(count[$index])) count[$index] = {}
      if (column.cascader) this.handleChange(column, row)
      if (!count[$index][column.prop]) {
        if (typeof column.change === 'function' && column.cell === true) {
          column.change({ row, column, index: $index, value: row[column.prop] })
        }
        this.grid.$emit('column-change', { row, column, index: $index, value: row[column.prop] })
        count[$index][column.prop] = true
        this.$nextTick(() => (count[$index][column.prop] = false))
      }
    },
    handleChange(column, row) {
      this.$nextTick(() => {
        const columnOption = [...this.grid.propOption]
        //本节点;
        const cascader = column.cascader
        const str = cascader.join(',')
        const columnNextProp = cascader[0]
        const value = row[column.prop]
        const rowIndex = row.$index
        // 下一个节点
        const columnNext = this.findObject(this.columnOption, columnNextProp)
        if (validatenull(columnNext)) return
        // 如果本节点没有字典则创建节点数组
        if (validatenull(this.grid.cascaderDIC[rowIndex])) {
          this.$set(this.grid.cascaderDIC, rowIndex, {})
        }
        if (this.grid.formIndexList.includes(rowIndex)) {
          //清空子类字典
          cascader.forEach((ele) => {
            this.$set(this.grid.cascaderDIC[rowIndex], ele.prop, [])
            cascader.forEach((ele) => (row[ele] = ''))
          })
        }
        //最后一级
        if (validatenull(cascader) || validatenull(value) || validatenull(columnNext)) {
          return
        }
        sendDic({
          column: columnNext,
          value: value,
          form: row,
        }).then((res) => {
          //首次加载的放入队列记录
          if (!this.grid.formIndexList.includes(rowIndex)) this.grid.formIndexList.push(rowIndex)
          const dic = Array.isArray(res) ? res : []
          // 修改字典
          this.$set(this.grid.cascaderDIC[rowIndex], columnNextProp, dic)

          if (
            !validatenull(dic[columnNext.cascaderIndex]) &&
            !validatenull(dic) &&
            !validatenull(columnNext.cascaderIndex)
          ) {
            row[columnNextProp] = dic[columnNext.cascaderIndex][(columnNext.props || {}).value || DIC_PROPS.value]
          }
        })
      })
    },
    openImg(list, index) {
      list = list.map((ele) => {
        return { thumbUrl: ele, url: ele }
      })
      this.$ImagePreview(list, index)
    },
    corArray(list, separator = DIC_SPLIT) {
      if (validatenull(list)) {
        return []
      } else if (!Array.isArray(list)) {
        return list.split(separator)
      }
      return list
    },
    handleDetail(row, column) {
      let result = row[column.prop]
      let DIC = column.parentProp ? (this.grid.cascaderDIC[row.$index] || {})[column.prop] : this.grid.DIC[column.prop]
      result = detail(row, column, this.grid.tableOption, DIC)
      if (!validatenull(DIC)) {
        row['$' + column.prop] = result
      }
      return result
    },
    getImgList(row, column) {
      let url = (column.propsHttp || {}).home || ''
      let value = (column.props || {}).value || DIC_PROPS.value
      if (validatenull(row[column.prop])) return []
      if (column.listType == 'picture-img') return [url + row[column.prop]]
      let list = this.corArray(this.deepClone(row[column.prop]), column.separator)
      list.forEach((ele, index) => {
        if (typeof ele === 'object') {
          list[index] = url + ele[value]
        } else {
          list[index] = url + ele
        }
      })
      return list
    },
  },
}
</script>

<style></style>
