<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form" layout="inline">
        <a-form-item
          label="体检卡号">
          <a-input placeholder="请输入体检卡号" v-decorator="['cardNumber', validatorRules.cardNumber ]"/>
        </a-form-item>
        <a-form-item
          label="体检人姓名">
          <a-input placeholder="请输入体检人姓名" v-decorator="['patientName', validatorRules.patientName ]" />
        </a-form-item>
        <a-form-item
          label="性别">
          <a-radio-group  buttonStyle="solid" v-decorator="[ 'patientSex', {}]">
            <a-radio-button :value="0">未知</a-radio-button>
            <a-radio-button :value="1">男</a-radio-button>
            <a-radio-button :value="2">女</a-radio-button>
          </a-radio-group>
        </a-form-item>
        <a-form-item
          label="生日">
          <a-date-picker showTime format='YYYY-MM-DD' v-decorator="[ 'patientBirthday', {}]" />
        </a-form-item>
        <a-form-item
          label="身份证号">
          <a-input placeholder="请输入身份证号" v-decorator="['idNumber', {}]" />
        </a-form-item>
        <a-form-item
          label="民族">
<!--          <a-input placeholder="请输入民族" v-decorator="['nation', {}]" />-->
          <j-dict-select-tag placeholder="请选择民族" v-decorator="['nation', {}]" :searchAble="true" :triggerChange="true" title="民族" dictCode="chinese_nation" style="width: 100px"/>
        </a-form-item>
        <a-form-item
          label="电子邮件">
          <a-input placeholder="请输入电子邮件" v-decorator="['email', {}]" />
        </a-form-item>
        <a-form-item
          label="电话">
          <a-input placeholder="请输入电话" v-decorator="['phone', {}]" />
        </a-form-item>
        <a-form-item
          label="籍贯">
          <a-input placeholder="请输入籍贯" v-decorator="['nativePlace', {}]" />
        </a-form-item>
        <a-form-item
          label="婚姻状况">
          <a-radio-group  buttonStyle="solid" v-decorator="[ 'maritalStatus', {}]">
            <a-radio-button :value="0">未知</a-radio-button>
            <a-radio-button :value="1">未婚</a-radio-button>
            <a-radio-button :value="2">已婚</a-radio-button>
          </a-radio-group>
        </a-form-item>
        <a-form-item
          label="照片">
          <a-input placeholder="请输入照片" v-decorator="['photo', {}]" />
        </a-form-item>
        <a-form-item
          label="邮编">
          <a-input placeholder="请输入邮编" v-decorator="['postcode', {}]" />
        </a-form-item>
        <a-form-item
          label="联系地址">
          <a-input placeholder="请输入联系地址" v-decorator="['contactAddress', {}]" />
        </a-form-item>
        <a-form-item
          label="单位编码">
          <a-input placeholder="请输入单位编码" v-decorator="['companyCode', {}]" />
        </a-form-item>
        <a-form-item
          label="部门编码">
          <a-input placeholder="请输入部门编码" v-decorator="['departCode', {}]" />
        </a-form-item>
        <a-form-item
          label="备注">
          <a-input placeholder="请输入备注" v-decorator="['remarks', {}]" />
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "HisPhysicalExaminationPatientModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        cardNumber:{rules: [{ required: true, message: '请输入体检卡号!' }]},
        patientName:{rules: [{ required: true, message: '请输入体检人姓名!' }]},
        registerBy:{rules: [{ required: true, message: '请输入登记人!' }]},
        registerTime:{rules: [{ required: true, message: '请输入登记时间!' }]},
        },
        url: {
          add: "/his/examination/hisPhysicalExaminationPatient/add",
          edit: "/his/examination/hisPhysicalExaminationPatient/edit",
        },
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'cardNumber','patientName','patientSex','idNumber','nation','email','phone','nativePlace','maritalStatus','photo','postcode','contactAddress','companyCode','departCode','remarks'));
		      //时间格式化
          this.form.setFieldsValue({patientBirthday:this.model.patientBirthday?moment(this.model.patientBirthday):null});
        });
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
              method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            formData.patientBirthday = formData.patientBirthday?formData.patientBirthday.format('YYYY-MM-DD'):null;

            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })
          }
        })
      },
      handleCancel () {
        this.close()
      }
    }
  }
</script>

<style lang="less" scoped>

</style>