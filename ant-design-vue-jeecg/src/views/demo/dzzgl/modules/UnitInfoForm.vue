<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="单位全称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['unitFullName']" placeholder="请输入单位全称"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="单位简称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['unitBrief']" placeholder="请输入单位简称"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="法人单位" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['izLawUnit']" :trigger-change="true" dictCode="yn12" placeholder="请选择法人单位"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="单位属性" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['unitType']" :trigger-change="true" dictCode="unit_type" placeholder="请选择单位属性"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="单位隶属关系" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['unitBelong']" :trigger-change="true" dictCode="unit_belong" placeholder="请选择单位隶属关系"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="联系电话" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['unitPhone', validatorRules.unitPhone]" placeholder="请输入联系电话"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="单位员工数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['unitStaffNum', validatorRules.unitStaffNum]" placeholder="请输入单位员工数量" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="单位负责人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['unitChief']" placeholder="请输入单位负责人"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="单位住址" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['unitAddr']" placeholder="请输入单位住址"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="单位介绍" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-editor v-decorator="['unitIntro',{trigger:'input'}]"/>
            </a-form-item>
          </a-col>
          <a-col v-if="showFlowSubmitButton" :span="24" style="text-align: center">
            <a-button @click="submitForm">提 交</a-button>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JFormContainer from '@/components/jeecg/JFormContainer'
  import JDictSelectTag from "@/components/dict/JDictSelectTag"
  import JEditor from '@/components/jeecg/JEditor'

  export default {
    name: 'UnitInfoForm',
    components: {
      JFormContainer,
      JDictSelectTag,
      JEditor,
    },
    props: {
      //流程表单data
      formData: {
        type: Object,
        default: ()=>{},
        required: false
      },
      //表单模式：true流程表单 false普通表单
      formBpm: {
        type: Boolean,
        default: false,
        required: false
      },
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        form: this.$form.createForm(this),
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
          unitPhone: {
            rules: [
              { required: false},
              { pattern: /^-?\d+\.?\d*$/, message: '请输入数字!'},
            ]
          },
          unitStaffNum: {
            rules: [
              { required: false},
              { pattern: /^-?\d+\.?\d*$/, message: '请输入数字!'},
            ]
          },
        },
        url: {
          add: "/dzzgl/unitInfo/add",
          edit: "/dzzgl/unitInfo/edit",
          queryById: "/dzzgl/unitInfo/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return false
          }
          return true
        }
        return this.disabled
      },
      showFlowSubmitButton(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return true
          }
        }
        return false
      }
    },
    created () {
      //如果是流程中表单，则需要加载流程表单data
      this.showFlowData();
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
          this.form.setFieldsValue(pick(this.model,'unitFullName','unitBrief','izLawUnit','unitType','unitBelong','unitPhone','unitStaffNum','unitChief','unitAddr','unitIntro'))
        })
      },
      //渲染流程表单数据
      showFlowData(){
        if(this.formBpm === true){
          let params = {id:this.formData.dataId};
          getAction(this.url.queryById,params).then((res)=>{
            if(res.success){
              this.edit (res.result);
            }
          });
        }
      },
      submitForm () {
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
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
         
        })
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'unitFullName','unitBrief','izLawUnit','unitType','unitBelong','unitPhone','unitStaffNum','unitChief','unitAddr','unitIntro'))
      },
    }
  }
</script>