<template>
  <div>
    <!-- <zg-table v-loading='loading' :initpage="initpage" :column="tableData.column" :data="tableData.data"  @change="handleChange" >
    </zg-table> -->
     <!-- label-width="80px" -->
    <zg-form ref="zgForm" :schema="schema" label-width="80px" @validate="validate"></zg-form>
  </div>
</template>

<script>
// import ZgTable from './zg-table/zg-table'
import ZgForm from './zg-form'
export default {
  name: 'HelloWorld',
  components: {
    // ZgTable,
    ZgForm
  },
  data (vm) {
    var validatePass = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密码'))
      } else {
        if (this.formData.checkPass !== '') {
          this.$refs.zgForm.validateField('checkPass')
        }
        callback()
      }
    }
    var validatePass2 = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.formData.pass) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }
    return {
      schema: [
        {
          type: 'input',
          vModel: 'pass',
          config: {
            type: 'password'
          },
          rules: [
            { validator: validatePass, trigger: 'blur' }
          ],
          label: '输入密码'
        },
        {
          type: 'input',
          vModel: 'checkPass',
          config: {
            type: 'password'
          },
          rules: [
            { validator: validatePass2, trigger: 'blur' }
          ],
          label: '确认密码'
        },
        {
          type: 'input',
          vModel: 'name',
          rules: [
            { required: true, message: '请输入活动名称', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ],
          label: '活动名称'
        },
        {
          type: 'select',
          config: {
            style: {
              width: '100%'
            },
            multiple: true,
            on: {
              change: (value) => {
                console.log(value)
              }
            }
          },
          vModel: 'regon',
          options: ['区域一', '区域二'],
          required: true,
          label: '活动区域',
          default: []
        },
        {
          type: 'select',
          config: {
            style: {
              width: '100%'
            }
          },
          vModel: 'regon1',
          options: [{ label: '区域一', value: 1 }, { label: '区域二', value: 2 }],
          required: true,
          label: '活动区域'
        },
        {
          render (h, { formData, context }) {
            return <div>
              <el-col span={11}>
                <el-form-item prop="date1">
                  <el-date-picker type="date" placeholder="选择日期" on-input={value => context.$set(formData, 'date1', value)} value={formData.date1} style="width: 100%;"></el-date-picker>
                </el-form-item>
              </el-col>
              <el-col class="line" style="text-align:center" span={2}>-</el-col>
              <el-col span={11}>
                <el-form-item prop="date2">
                  <el-date-picker type="date" placeholder="选择日期" on-input={value => context.$set(formData, 'date2', value)} value={formData.date2} style="width: 100%;"></el-date-picker>
                </el-form-item>
              </el-col>
            </div>
          },
          rules: {
            date1: [{ required: true, message: '选择日期', trigger: 'blur' }],
            date2: [{ required: true, message: '选择时间', trigger: 'blur' }]
          },
          label: '活动时间'
        },
        {
          type: 'switch',
          vModel: 'delivery',
          label: '即时配送'
        },
        {
          type: 'radio-group',
          vModel: 'type',
          options: ['美食/餐厅线上活动', '地推活动', '线下主题活动', '单纯品牌曝光'],
          label: '活动性质'
        },
        {
          type: 'checkbox-group',
          vModel: 'checkbox',
          options: ['区域一', '区域二'],
          required: true,
          label: '活动区域'
        },
        {
          type: 'input',
          config: {
            type: 'textarea'
          },
          vModel: 'desc',
          label: '活动形式'
        },
        {
          render (h, { that, formData, context }) {
            return <div>
              <el-button type="primary" on-click={() => {
                context.resetFields()
                // context.validate((valid) => {
                //   if (valid) {
                //     console.log(context.formData)

                //     alert('submit!')
                //   } else {
                //     console.log('error submit!!')
                //     return false
                //   }
                // })
              }}>提交</el-button>
              <el-button on-click={() => {
                context.resetFields()
              }} >重置</el-button>
            </div>
          }
        }
      ]
    }
  },
  computed: {
    formData () {
      return this.$refs.zgForm.formData
    }
  },
  methods: {
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          alert('submit!')
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    resetForm (formName) {
      this.$refs[formName].resetFields()
    }
  }
}
</script>
