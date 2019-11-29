<template>
  <!-- <el-form ref="form" class="custom-form" v-bind="$attrs">
    <slot name="title" />
    <el-form-item v-for="(field, index) in schema" v-show="!field.hidden" :key="index" class="custom-form-item" v-bind="filterFormItemAttrs(field)">
      <component :is="field.type?'el-'+field.type:field" v-model="field[field.vModel]" :default-first-option="true" v-bind="filterComponentAttrsfield(field)" clearable v-on="resetEvent(field)">
        <span v-if="field.slotDefault" v-html="field.slotDefault" />
        <component :is="field.slot" v-if="field.slot" />
        <template v-if="field.type === 'select'">
          <el-option
            v-for="(item,i) in field.options||[]"
            :key="i"
            :disabled="item.disabled"
            :label="handleLabel(item,field)"
            :value="handleValue(item,field,i)"
          />
        </template>
        <template v-if="field.type === 'radio-group'">
          <el-radio
            v-for="(item,i) in field.options"
            :key="i"
            :disabled="item.disabled"
            :label="handleValue(item,field,i)"
          >{{ handleLabel(item,field,i) }}</el-radio>
        </template>
      </component>
      <el-button v-if="field.append" :type="field.append.type || 'pramiry'" :disabled="field.append.disabled" :style="field.append.style" class="custom-input-append" @click="e=>field.append.click && field.append.click(e,field.append)">{{ field.append.text }}</el-button>
    </el-form-item>
  </el-form> -->
  <el-form ref="el-form" v-bind="$attrs" :rules="rules" :model="formData" v-on='$listeners'>
    <el-form-item v-for="(item, index) in schema" v-show="!item.hidden" :key="index" :prop="item.vModel" v-bind="handleItemBind(item)">
      <el-radio-group v-if="item.type === 'radio-group'" v-model="formData[item.vModel]">
        <el-radio :label="e" v-for="(e,i) in item.options||[]" :key="i+'_'+index"></el-radio>
      </el-radio-group>
      <component v-else-if="item.type" :is="`el-${item.type}`" v-model="formData[item.vModel]" :key="index+item.type" v-on="(item.config||{}).on||{}" v-bind="handleItemConfigBind(item.config)">
        <template v-if="item.type === 'select'">
          <el-option
            v-for="(e,i) in item.options||[]"
            :key="i+'_'+index"
            :disabled="item.disabled"
            :label="handleLabel(e,item)"
            :value="handleValue(e,item,i)"
          />
        </template>
        <template v-if="item.type === 'radio-group'">
          <el-radio :label="e" v-for="(e,i) in item.options||[]" :key="i+'_'+index"></el-radio>
        </template>
        <template v-if="item.type === 'checkbox-group'">
          <el-checkbox :label="e" v-for="(e,i) in item.options||[]" :key="i+'_'+index"></el-checkbox>
        </template>
      </component>
      <zg-render
        v-else-if="item.render"
        :scope="{formData,that:item,context}"
        :render="item.render">
        </zg-render>
    </el-form-item>
    <el-form-item>
      {{formData}}
    </el-form-item>
  </el-form>
</template>

<script>
import ZgRender from './zg-render'
export default {
  name: 'ComForm',
  components: {
    ZgRender
  },
  props: {
    schema: {
      type: Array,
      required: true
    }
  },
  data (vm) {
    const rules = {}
    const formData = vm.schema.reduce((prev, curr) => {
      if (curr.required)rules[curr.vModel] = [{ required: true, trigger: 'blur' }]
      if (curr.rules) {
        if (Array.isArray(curr.rules))rules[curr.vModel] = curr.rules
        else {
          Object.keys(curr.rules).forEach(key => {
            prev[key] = curr.rules[key].default
            rules[key] = curr.rules[key]
          })
        }
      }
      curr.vModel && (prev[curr.vModel] = curr.type === 'checkbox-group' ? curr.default || [] : curr.default)
      return prev
    }, {})
    return {
      showMessage: false,
      formData,
      rules
    }
  },
  computed: {
    context () {
      return this
    }
  },
  mounted () {
    const { validate, validateField, resetFields, clearValidate } = this.$refs['el-form']
    this.schema.forEach((item, index) => {
      const { resetField, clearValidate } = this.$refs['el-form'].$children[index]
      Object.assign(item, { resetField, clearValidate })
    })
    Object.assign(this, { validate, validateField, resetFields, clearValidate })
  },
  methods: {
    handleItemBind (item) {
      const { config, rules, resetField, clearValidate, ...data } = item
      return rules ? { required: true, ...data } : data
    },
    handleItemConfigBind (config = {}) {
      const { on, ...data } = config
      return data
    },

    verifyAll () {
      const obj = {}
      this.schema.forEach(item => {
        // if (typeof item[item.vModel] === "string")
        //   item[item.vModel] = item[item.vModel].tirm();
        if (item.required && isEmpty(item[item.vModel]) && item[item.vModel] !== 0) item.error = item.errorMassage || '此项必填'
        else if (item.validate && !item.validate(item)) item.error = item.errorMassage || '不合规则'
        else item.error = ''
        obj[item.vModel] = item[item.vModel]
      })
      const dataIsOk = !this.schema.some(item => !!item.error)
      delete obj.undefined
      return dataIsOk && obj
    },
    initForm () {
      this.schema.splice(this._initValues.length, this.schema.length - this._initValues.length)
      this.schema.forEach((item, index) => {
        item[item.vModel] = this._initValues[index][1]
        item.hidden = this._initValues[index][0]
        item.error = ''
      })
    },
    handleInput (value, item) {
      if (item.required && !item[item.vModel]) item.error = item.errorMassage
      else item.error = ''
      item.onInput && item.onInput(value, item)
    },
    handleChange ($event, field, index) {
      const found = Array.isArray(field.options) && field.options.find(item => item.name === $event)
      field.onChange && field.onChange(field, $event, found, index)
    },
    // handleLabel(item, field) {
    //   if (labelOrValue === 'index') return index
    //   return typeof item === 'object' ? (labelOrValue ? item[labelOrValue] : item.name) : item
    // },
    handleValue (item, field, index) {
      const { optValue, valueKey } = field
      if (optValue === 'index') return index
      else if (valueKey) return item
      else { return typeof item === 'object' ? (optValue ? item[optValue] : item.value) : item }
    },
    handleLabel (item, field, index) {
      const { optLabel } = field
      return typeof item === 'object' ? (optLabel ? item[optLabel] : item.label) : item
    },
    handleControl (controlBy) {
      if (!controlBy || isEmpty(controlBy)) return true
      const found = this.schema.find(item => item.vModel === controlBy.name)
      return found ? (controlBy.handle && controlBy.handle(found[found.vModel])) : true
    },

    filterFormItemAttrs (field) {
      const { label, required, error, formItemStyle } = field
      return { label, required, error, style: formItemStyle }
    },
    filterComponentAttrsfield (field) {
      return field.type ? field : {}
    },
    resetEvent (field) {
      const res = {}
      if (field.on) Object.keys(field.on).forEach(key => (res[key] = (...arg) => field.on[key].apply(null, [{ elementArgs: arg, field, context: this }])))
      if (field.type === 'input' && !(field.on || {}).input) {
        res.input = value => {
          if (field.required && !field[field.vModel]) field.error = field.errorMassage
          else field.error = ''
        }
      }
      return res
    }
  }
}
</script>
<style scoped lang="scss">
.custom-form{
  .custom-form-item{
    .custom-input-append{
      position: absolute;
      top: 0px;
      right: 0px;
      width: 112px;
      border: solid 1px #ddd;
      border-radius: 0 4px 4px 0;
    }
  }
}
// .com-form {
//   position: relative;
//   &.form-flex {
//     display: flex;
//     flex-wrap: wrap;
//     & > div {
//       width: 33%;
//     }
//   }
//   & > .btn-box {
//     width: 100% !important;
//     display: flex;
//     justify-content: flex-end;
//   }
//   /deep/ .el-transfer-panel {
//     width: 38%;
//   }
//   /deep/ .el-form-item__label {
//     white-space: nowrap;
//     text-overflow:ellipsis;
//     overflow: hidden;
//   }
// }
</style>
