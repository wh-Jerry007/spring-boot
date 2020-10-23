<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label=" 姓名" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['name']" placeholder="请输入 姓名"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label=" 性别" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['sex']" :trigger-change="true" dictCode="gender" placeholder="请选择 性别"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="民族" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['gendernation']" :trigger-change="true" dictCode="nation" placeholder="请选择民族"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label=" 出生日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择 出生日期" v-decorator="['nationbirthday']" :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="学历" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['eduHostory']" :trigger-change="true" dictCode="education" placeholder="请选择学历"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="组织名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['educationpartyName']" placeholder="请输入组织名称"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="申请入党日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择申请入党日期" v-decorator="['partyNamepartyDate']" :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label=" 成为积极分子日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择 成为积极分子日期" v-decorator="['partyJjfzDate']" :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label=" 成为发展对象日期" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择 成为发展对象日期" v-decorator="['partyFzdxDate']" :trigger-change="true" style="width: 100%"/>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="手机号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['partyTel', validatorRules.partyTel]" placeholder="请输入手机号"></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="党员类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['partyType']" :trigger-change="true" dictCode="party_member_type" placeholder="请选择党员类型"/>
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

  export default {
    name: 'DyfzglFzdxForm',
    components: {
      JFormContainer,
      JDate,
      JDictSelectTag,
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
          partyTel: {
            rules: [
              { required: false},
              { pattern: /^1[3456789]\d{9}$/, message: '请输入正确的手机号码!'},
            ]
          },
        },
        url: {
          add: "/fzdx/dyfzglFzdx/add",
          edit: "/fzdx/dyfzglFzdx/edit",
          queryById: "/fzdx/dyfzglFzdx/queryById"
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
          this.form.setFieldsValue(pick(this.model,'name','sex','gendernation','nationbirthday','eduHostory','educationpartyName','partyNamepartyDate','partyJjfzDate','partyFzdxDate','partyTel','partyType'))
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
        this.form.setFieldsValue(pick(row,'name','sex','gendernation','nationbirthday','eduHostory','educationpartyName','partyNamepartyDate','partyJjfzDate','partyFzdxDate','partyTel','partyType'))
      },
    }
  }
</script>