<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-model-item label="提单号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="billnum">
              <a-input v-model="model.billnum" placeholder="请输入提单号"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="type">
              <a-input v-model="model.type" placeholder="请输入类型"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="供应商" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="supplier">
              <a-input v-model="model.supplier" placeholder="请输入供应商"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="承运商" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="arriage">
              <a-input v-model="model.arriage" placeholder="请输入承运商"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="清关公司" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="customsClearance">
              <a-input v-model="model.customsClearance" placeholder="请输入清关公司"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="路线" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="line">
              <a-input v-model="model.line" placeholder="请输入路线"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="用户" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="user">
              <a-input v-model="model.user" placeholder="请输入用户"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="箱数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="boxes">
              <a-input-number v-model="model.boxes" placeholder="请输入箱数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="合箱数" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="allBoxes">
              <a-input-number v-model="model.allBoxes" placeholder="请输入合箱数" style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="实重" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="weight">
              <a-input v-model="model.weight" placeholder="请输入实重"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="材重" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="materialWeight">
              <a-input v-model="model.materialWeight" placeholder="请输入材重"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="收费重" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="heavyCharge">
              <a-input v-model="model.heavyCharge" placeholder="请输入收费重"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="体积" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="volume">
              <a-input v-model="model.volume" placeholder="请输入体积"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="盈亏" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="profit">
              <a-input v-model="model.profit" placeholder="请输入盈亏"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
              <a-input v-model="model.remark" placeholder="请输入备注"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="内部备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="inRemark">
              <a-input v-model="model.inRemark" placeholder="请输入内部备注"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="出发时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="departure">
              <j-date placeholder="请选择出发时间" v-model="model.departure"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="到达时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="arrive">
              <j-date placeholder="请选择到达时间" v-model="model.arrive"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="12">
            <a-form-model-item label="清关时间" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="customsClearanceTime">
              <j-date placeholder="请选择清关时间" v-model="model.customsClearanceTime"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'ZmBillloadingForm',
    components: {
    },
    props: {
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
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
          add: "/zmexpress/zmBillloading/add",
          edit: "/zmexpress/zmBillloading/edit",
          queryById: "/zmexpress/zmBillloading/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
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
      submitForm () {
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
            httpAction(httpurl,this.model,method).then((res)=>{
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
    }
  }
</script>