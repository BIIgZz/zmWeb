<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form-model ref="form" :model="model" :rules="validatorRules">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="公司名" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="company">
              <a-input v-model="model.company"placeholder="请输入公司名" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="地址" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="address">
              <a-input v-model="model.address"placeholder="请输入地址" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="公司营业执照号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="licenseNumber">
              <a-input v-model="model.licenseNumber"placeholder="请输入公司营业执照号" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="法人身份证" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="idCard">
              <a-input v-model="model.idCard"placeholder="请输入法人身份证" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="法人代表" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="legalRepresentative">
              <a-input v-model="model.legalRepresentative"placeholder="请输入法人代表" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="法人电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="lrTel">
              <a-input v-model="model.lrTel"placeholder="请输入法人电话" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="业务联系人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="businessContact">
              <a-input v-model="model.businessContact"placeholder="请输入业务联系人" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="业务联系人电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="businessContactTel">
              <a-input v-model="model.businessContactTel"placeholder="请输入业务联系人电话" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="操作" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="operate">
              <a-input v-model="model.operate"placeholder="请输入操作" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="操作电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="operateTel">
              <a-input v-model="model.operateTel"placeholder="请输入操作电话" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="财务" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="finance">
              <a-input v-model="model.finance"placeholder="请输入财务" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="财务电话" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="financeTel">
              <a-input v-model="model.financeTel"placeholder="请输入财务电话" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="财务QQ" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="financeQq">
              <a-input v-model="model.financeQq"placeholder="请输入财务QQ" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="其他" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="other">
              <a-input v-model="model.other"placeholder="请输入其他" ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="营业执照附件" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="licenseAttachment">
              <j-upload v-model="model.licenseAttachment"  ></j-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="公司类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="type">
              <j-dict-select-tag type="list" v-model="model.type" dictCode="zm_supplier,type,type" placeholder="请选择公司类型" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </a-spin>
  </j-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: "ZmPartnerModal",
    components: {
    },
    props:{
      mainId:{
        type:String,
        required:false,
        default:''
      }
    },
    data () {
      return {
        title:"操作",
        width:800,
        visible: false,
        model:{
        },
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
          add: "/zmexpress/zmSupplier/addZmPartner",
          edit: "/zmexpress/zmSupplier/editZmPartner",
        }

      }
    },
    created () {
    //备份model原始值
      this.modelDefault = JSON.parse(JSON.stringify(this.model));
    },
    methods: {
      add () {
        this.edit(this.modelDefault);
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
      },
      close () {
        this.$emit('close');
        this.visible = false;
        this.$refs.form.clearValidate();
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
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
            this.model['type'] = this.mainId
            httpAction(httpurl,this.model,method).then((res)=>{
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
          }else{
             return false
          }
        })
      },
      handleCancel () {
        this.close()
      },


    }
  }
</script>
