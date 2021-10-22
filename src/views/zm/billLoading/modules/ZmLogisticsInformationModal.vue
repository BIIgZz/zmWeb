<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    switchFullscreen
    @ok="handleOk"
    :okButtonProps="{ class:{'jee-hidden': disableSubmit} }"
    @cancel="handleCancel"
    cancelText="关闭">
    <zm-logistics-information-form ref="realForm" @ok="submitCallback" :disabled="disableSubmit"></zm-logistics-information-form>
  </j-modal>
</template>

<script>

  import ZmLogisticsInformationForm from './ZmLogisticsInformationForm'
  export default {
    name: 'ZmLogisticsInformationModal',
    components: {
      ZmLogisticsInformationForm
    },
    data () {
      return {
        title:'',
        width:800,
        visible: false,
        disableSubmit: false
      }
    },
    methods: {
      add (billDtail) {
        this.visible=true
        this.$nextTick(()=>{
          this.$refs.realForm.add(billDtail);
        })
      },
      edit (record) {
        this.visible=true
        this.$nextTick(()=>{
          this.$refs.realForm.edit(record);
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        this.$refs.realForm.submitForm();
      },
      submitCallback(){
        this.$emit('ok');
        this.visible = false;
      },
      handleCancel () {
        this.close()
      }
    }
  }
</script>