<template>
  <div>
    <van-cell is-link
              @click="showSelect = !showSelect,err = false"
              :title="info.chineseName"
              :required="info.empty === 0"
              :value="formItem.input">
      <span v-if="err"
            slot="right-icon"
            class="err-tip">请选择{{info.chineseName}}
        <van-icon name="arrow" />
      </span>
    </van-cell>
    <van-popup position="bottom"
               v-model="showSelect">
      <van-picker :defaultIndex="defaultIndex"
                  show-toolbar
                  :title="info.chineseName"
                  @cancel="onCancel"
                  @confirm="onConfirm"
                  :columns="columns" />
    </van-popup>
  </div>
</template>
<script>
export default {
  props: {
    isEdit: {
      type: Boolean
    },
    editVal: {
      type: Object
    },
    info: {
      type: Object
    },
    foreignKeyValues: {
      type: Object
    },
    defaultData: {
      type: Object
    }
  },
  data () {
    return {
      formItem: {
        input: ''
      },
      err: false,
      columns: [],
      showSelect: false
    }
  },
  watch: {
    editVal (v) {
      console.log(this.foreignKeyValues, v[this.info.englishName])
      this.formItem.input = v[this.info.englishName] ? v[this.info.englishName].displayValue : ''
    }
  },
  created () {
    if (this.defaultData && this.info.englishName in this.defaultData) {
      if (Array.isArray(this.defaultData[this.info.englishName])) {
        this.columns = (this.foreignKeyValues[this.info.englishName] || []).filter(
          item => this.defaultData[this.info.englishName].includes(item.id)
        ).map(i => i.displayValue)
      } else {
        this.columns = [(this.foreignKeyValues[this.info.englishName] || []).find(
          item => item.id === this.defaultData[this.info.englishName]
        ).displayValue]
        this.formItem.input = this.columns[0]
        this.disabled = true
      }
    } else {
      this.columns = (this.foreignKeyValues[this.info.englishName] || []).map(
        item => item.displayValue
      )
    }
  },
  computed: {
    defaultIndex () {
      return this.columns.indexOf(this.formItem.input)
    }
  },
  methods: {
    async sendVal () {
      if (this.info.empty === 0 && this.formItem.input.trim() === '') {
        this.err = true
        return false
      }
      if (this.info.uniqued === 1 && !this.isEdit) {
        let data = {
          tableId: this.tableConfig.id, // [必填]表单主键ID，由当面所在表单查询页面维护
          tableName: this.tableConfig.englishName, // [必填]表单配置的表名称
          columnName: this.info.englishName, // [必填]判断重复的字段名
          columnValue: this.formItem.input.trim()
        }
        let res = await uniquedData(data)
        if (res.data.code === 500) {
          this.$notify({
            message: res.data.message,
            duration: 1000,
            background: '#f44'
          })
          this.err = true
          return false
        } else {
          return {
            [this.info.englishName]: this.foreignKeyValues[this.info.englishName].find(item => item.displayValue === this.formItem.input).id
          }
        }
      }
      return {
        [this.info.englishName]: this.foreignKeyValues[this.info.englishName].find(item => item.displayValue === this.formItem.input).id
      }
    },
    onCancel () {
      this.showSelect = false
    },
    onConfirm (val, v2) {
      this.formItem.input = val
      this.showSelect = false
    }
  }
}
</script>
<style scoped>
.err-tip {
  height: 24px;
  font-size: 14px;
  color: #f44;
  line-height: 24px;
}
.err-tip .van-icon {
  font-size: 16px;
  vertical-align: middle;
}
</style>
