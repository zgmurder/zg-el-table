<template>
  <div>
    <zg-table v-loading='loading' :initpage="initpage" :column="tableData.column" :data="tableData.data"  @change="handleChange" >
    </zg-table>
  </div>
</template>

<script>
import ZgTable from './zg-table/zg-table'
export default {
  name: 'HelloWorld',
  components: {
    ZgTable
  },
  data () {
    return {
      initpage: {
        currentPage: 1,
        pageSizes: [6, 10, 15],
        pageSize: 6,
        background: true,
        prevText: '哈哈哥',
        style: {
          textAlign: 'left',
          marginTop: '20px'
        }
      },
      tableData: {
        column: [
          {
            prop: 'date',
            label: '日期'
          },
          {
            prop: 'page',
            label: '页码'
          },
          {
            prop: 'province',
            label: '省份'
          },
          {
            prop: 'city',
            label: '市区'
          },
          {
            prop: 'address',
            label: '地址'
          }
        ],
        data: [
          // {
          //   date: '2016-05-03',
          //   name: '王小虎1',
          //   province: '上海',
          //   city: '普陀区',
          //   address: '上海市普陀区金沙江路 1518 弄',
          //   zip: 200333
          // },
        ]
      },
      loading: true
    }
  },
  methods: {
    handleChange (pageConfig) {
      this._temPageConfig = pageConfig || this._temPageConfig
      if (!pageConfig) this._temPageConfig.init()
      const { pageSize, currentPage } = this._temPageConfig
      this.loading = true
      this.getHistory({ page: currentPage, size: pageSize }).then(res => {
        this.tableData.data = res.data
        this._temPageConfig.total = res.total
        this.loading = false
      })
    },
    getHistory (option) {
      const { page, size } = option
      const data = Array.from({ length: size }).map(item => ({
        date: '2016-05-03',
        name: '王小虎1',
        province: '上海',
        city: '普陀区',
        address: '上海市普陀区金沙江路 1518 弄',
        page
      }))
      return new Promise((resolve, reject) => {
        setTimeout(() => {
          resolve({ total: 25, code: 200, data })
        }, 1000)
      })
    }
  }
}
</script>
