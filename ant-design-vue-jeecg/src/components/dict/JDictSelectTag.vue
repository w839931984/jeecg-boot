<template>
  <a-radio-group v-if="tagType=='radio'" @change="handleInput" :value="value" :disabled="disabled">
    <a-radio v-for="(item, key) in dictOptions" :key="key" :value="item.value">{{ item.text }}</a-radio>
  </a-radio-group>

  <a-select
    :showSearch="searchAble"
    v-else-if="tagType=='select'"
    :placeholder="placeholder"
    :disabled="disabled"
    :value="value"
    :filterOption="filterOption"
    @change="handleInput">
    <a-select-option value="">
      <span style="display: inline-block;width: 100%" title="请选择">
        请选择
      </span>
    </a-select-option>
    <a-select-option v-for="(item, key) in dictOptions" :key="key" :value="item.value">
      <span style="display: inline-block;width: 100%" :title=" item.text || item.label ">
        {{ item.text || item.label }}
      </span>
    </a-select-option>
  </a-select>
</template>

<script>
  import {ajaxGetDictItems} from '@/api/api'

  export default {
    name: "JDictSelectTag",
    props: {
      dictCode: String,
      placeholder: String,
      triggerChange: Boolean,
      disabled: Boolean,
      searchAble: Boolean,
      value: String,
      type: String,
      decorator: Object
    },
    data() {
      return {
        dictOptions: [],
        tagType:"",

      }
    },
    created() {
      this.pinyin = require("pinyin");
      // console.log(this.dictCode);
      // console.log(this.value);
      if(!this.type || this.type==="list"){
        this.tagType = "select"
      }else{
        this.tagType = this.type
      }
      //获取字典数据
      this.initDictData();
    },
    methods: {
      initDictData() {
        //根据字典Code, 初始化字典数组
        ajaxGetDictItems(this.dictCode, null).then((res) => {
          if (res.success) {
//                console.log(res.result);
            this.dictOptions = res.result;
          }
        })
      },
      handleInput(e) {
        let val;
        if(this.tagType=="radio"){
          val = e.target.value
        }else{
          val = e
        }
        //console.log(val);
        if(this.triggerChange){
          this.$emit('change', val);
        }else{
          this.$emit('input', val);
        }
      },
      setCurrentDictOptions(dictOptions){
        this.dictOptions = dictOptions
      },
      getCurrentDictOptions(){
        return this.dictOptions
      },
      filterOption(input, option) {
        let lowerInput = input.toLowerCase();
        let text = option.componentOptions.children[0].children[0].text.toLowerCase();
        let array = this.pinyin(text, {
          style: this.pinyin.STYLE_NORMAL
        });
        //首字母
        let initials = "";
        array.forEach(word =>{
          // console.log(word);
          initials += word[0].substr(0, 1);
        });
        // console.log(initials);
        return (text.indexOf(lowerInput) >= 0)
          || (initials.toLowerCase().indexOf(lowerInput) >= 0);
      }
    }
  }
</script>

<style scoped>
</style>