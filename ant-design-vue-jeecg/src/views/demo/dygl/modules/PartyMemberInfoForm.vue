<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-item label="党员姓名" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['memberName']" placeholder="请输入党员姓名"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="性别" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['sex']" :trigger-change="true" dictCode="gender" placeholder="请选择性别"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="民族" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['nation']" :trigger-change="true" dictCode="nation" placeholder="请选择民族"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="出生日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择出生日期" v-decorator="['birthday']" :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="学历" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['education']" :trigger-change="true" dictCode="education" placeholder="请选择学历"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="组织标识" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['partyOrgId']" placeholder="请输入组织标识"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="职务" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['duty']" :trigger-change="true" dictCode="position" placeholder="请选择职务"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="入党日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择入党日期" v-decorator="['joinDate']" :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="转正日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择转正日期" v-decorator="['formalDate']" :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="手机号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['mobilePhone']" placeholder="请输入手机号"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="党员类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['memberType']" :trigger-change="true" dictCode="party_member_type" placeholder="请选择党员类型"/>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="党组树选择" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-category-select v-decorator="['partyCategory']" pcode="" placeholder="请选择党组树选择" />
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
  import JDate from '@/components/jeecg/JDate'  
  import JDictSelectTag from "@/components/dict/JDictSelectTag"
  import JCategorySelect from '@/components/jeecg/JCategorySelect'

  export default {
    name: 'PartyMemberInfoForm',
    components: {
      JFormContainer,
      JDate,
      JDictSelectTag,
      JCategorySelect,
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
        },
        url: {
          add: "/dygl/partyMemberInfo/add",
          edit: "/dygl/partyMemberInfo/edit",
          queryById: "/dygl/partyMemberInfo/queryById"
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
          this.form.setFieldsValue(pick(this.model,'memberName','sex','nation','birthday','education','partyOrgId','duty','joinDate','formalDate','mobilePhone','memberType','partyCategory'))
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
        this.form.setFieldsValue(pick(row,'memberName','sex','nation','birthday','education','partyOrgId','duty','joinDate','formalDate','mobilePhone','memberType','partyCategory'))
      },
      handleCategoryChange(value,backObj){
        this.form.setFieldsValue(backObj)
      }
    }
  }
</script>