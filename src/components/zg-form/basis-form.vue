<template>
  <zg-form :schema="schema"  label-width="80px"></zg-form>
</template>

<script>
export default {
  name: 'BasisForm',
  data () {
    return {
      schema: [
        {
          type: 'input',
          vModel: 'name',
          label: '活动名称',
          rules:[
            { required: true, message: '请输入活动名称', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ]
        },
        {
          type: 'select',
          config: {
            style: {
              width: '100%'
            }
          },
          vModel: 'date1',
          options: ['区域一', '区域二'],
          required: true,
          label: '活动区域',
          message:'请输入活动名称'
        },
        {
          render (h,{formData}) {
            return <div>
              <el-col span={11}>
                <el-date-picker type="date" placeholder="选择日期" value={formData.date1} on-change={value=>formData.date1 = value} style="width: 100%;"></el-date-picker>
              </el-col>
              <el-col class="line" style="text-align:center" span={2}>-</el-col>
              <el-col span={11}>
                <el-time-picker placeholder="选择时间" value={formData.date2} on-change={value=>formData.date2 = value} style="width: 100%;"></el-time-picker>
              </el-col>
            </div>
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
                context.validate((valid) => {
                if (valid) {
                  alert('submit!');
                } else {
                  console.log('error submit!!');
                  return false;
                }});
              }}>提交</el-button>
              <el-button on-click={()=>{
                context.resetFields();
              }} >重置</el-button>
            </div>
          }
        }
      ]
    }
  },
}
</script>
