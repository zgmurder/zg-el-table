<template>
  <div class="zg-table">
    <el-table ref="elTable"
      v-bind="$attrs"
      v-on="$listeners"
      :data="data"
      :span-method="this.merge ? this.mergeMethod : this.spanMethod">
      <zg-column v-bind="$attrs"
        v-for="(item, index) in column"
        :key="index"
        :column="item">
      </zg-column>
    </el-table>
    <!-- <el-pagination class="zg-table-pagination"
      v-if="pagination"
      v-bind="$attrs"
      v-on="$listeners"
      @current-change="paginationCurrentChange"
      :style="{ 'margin-top': paginationTop, 'text-align': paginationAlign }">
    </el-pagination> -->
    <el-pagination
      v-if="!!pageConfig.total"
      :current-page="pageConfig.currentPage"
      :page-sizes="pageConfig.pageSizes"
      :page-size="pageConfig.pageSize"
      :layout="pageConfig.layout"
      :total="pageConfig.total"
      @size-change="pageConfig.handleSizeChange"
      @current-change="pageConfig.handleCurrentChange"
      v-bind="initpage"
    />
  </div>
</template>

<script>
import ZgColumn from './zg-column'
export default {
  props: {
    column: Array,
    data: Array,
    spanMethod: Function,
    initpage: Object,
    merge: Array
  },
  components: {
    ZgColumn
  },
  data (vm) {
    const { currentPage = 1, pageSizes, pageSize = 15, layout = `total, sizes, prev, pager, next, jumper` } = vm.initpage || {}
    const defaultValue = { currentPage, pageSizes, pageSize, layout }
    const pageConfig = {
      ...defaultValue,
      total: 0,
      handleSizeChange: pageSize => {
        if (pageSize) vm.pageConfig.pageSize = pageSize
        this.$emit('change', vm.pageConfig || pageConfig)
      },
      handleCurrentChange: currentPage => {
        if (currentPage) vm.pageConfig.currentPage = currentPage
        this.$emit('change', vm.pageConfig || pageConfig)
      },
      init: () => {
        Object.assign(vm.pageConfig, defaultValue)
      }
    }
    pageConfig.handleCurrentChange()

    return {
      mergeLine: {},
      mergeIndex: {},
      pageConfig
    }
  },

  created () {
    this.getMergeArr(this.data, this.merge)
  },
  computed: {
    dataLength () {
      return this.data.length
    }
  },
  methods: {
    clearSelection () {
      this.$refs.elTable.clearSelection()
    },
    toggleRowSelection (row, selected) {
      this.$refs.elTable.toggleRowSelection(row, selected)
    },
    toggleAllSelection () {
      this.$refs.elTable.toggleAllSelection()
    },
    toggleRowExpansion (row, expanded) {
      this.$refs.elTable.toggleRowExpansion(row, expanded)
    },
    setCurrentRow (row) {
      this.$refs.elTable.setCurrentRow(row)
    },
    clearSort () {
      this.$refs.elTable.clearSort()
    },
    clearFilter (columnKey) {
      this.$refs.elTable.clearFilter(columnKey)
    },
    doLayout () {
      this.$refs.elTable.doLayout()
    },
    sort (prop, order) {
      this.$refs.elTable.sort(prop, order)
    },
    paginationCurrentChange (val) {
      this.$emit('p-current-change', val)
    },
    getMergeArr (tableData, merge) {
      if (!merge) return
      this.mergeLine = {}
      this.mergeIndex = {}
      merge.forEach((item, k) => {
        tableData.forEach((data, i) => {
          if (i === 0) {
            this.mergeIndex[item] = this.mergeIndex[item] || []
            this.mergeIndex[item].push(1)
            this.mergeLine[item] = 0
          } else {
            if (data[item] === tableData[i - 1][item]) {
              this.mergeIndex[item][this.mergeLine[item]] += 1
              this.mergeIndex[item].push(0)
            } else {
              this.mergeIndex[item].push(1)
              this.mergeLine[item] = i
            }
          }
        })
      })
    },
    mergeMethod ({ row, column, rowIndex, columnIndex }) {
      const index = this.merge.indexOf(column.property)
      if (index > -1) {
        const _row = this.mergeIndex[this.merge[index]][rowIndex]
        const _col = _row > 0 ? 1 : 0
        return {
          rowspan: _row,
          colspan: _col
        }
      }
    }
  },
  watch: {
    merge () {
      this.getMergeArr(this.data, this.merge)
    },
    dataLength () {
      this.getMergeArr(this.data, this.merge)
    }
  }
}

</script>
