<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    :destroyOnClose="true"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">

        <a-form-item label="指标年度" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['indexYear']" placeholder="请输入指标年度"></a-input>
        </a-form-item>
        <a-form-item label="经营对象" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['businessObject']" placeholder="请输入经营对象"></a-input>
        </a-form-item>
        <a-form-item label="所在部门" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input v-decorator="['department']" placeholder="请输入所在部门"></a-input>
        </a-form-item>
        <a-form-item label="指标新签合同额(万元)" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="['newContract']" placeholder="请输入指标新签合同额(万元)" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="指标合同回款额(万元)" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-input-number v-decorator="['contractPayment']" placeholder="请输入指标合同回款额(万元)" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="提交时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-date placeholder="请选择提交时间" v-decorator="['submissionTime']" :trigger-change="true" style="width: 100%"/>
        </a-form-item>
        <a-form-item label="父级节点" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <j-tree-select
            ref="treeSelect"
            placeholder="请选择父级节点"
            v-decorator="['pid']"
            dict="jyryzbgl,index_year,id"
            pidField="pid"
            pidValue="0"
            hasChildField="has_child">
          </j-tree-select>
        </a-form-item>
        
      </a-form>
    </a-spin>
  </j-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'
  import JDate from '@/components/jeecg/JDate'  
  import JTreeSelect from '@/components/jeecg/JTreeSelect'
  
  export default {
    name: "JyryzbglModal",
    components: { 
      JDate,
      JTreeSelect
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
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
          add: "/estar/jyryzbgl/add",
          edit: "/estar/jyryzbgl/edit",
        },
        expandedRowKeys:[],
        pidField:"pid"
     
      }
    },
    created () {
    },
    methods: {
      add (obj) {
        this.edit(obj);
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'indexYear','businessObject','department','newContract','contractPayment','submissionTime','pid'))
        })
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
            let old_pid = this.model[this.pidField]
            let formData = Object.assign(this.model, values);
            let new_pid = this.model[this.pidField]
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                this.$emit('ok');
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
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'indexYear','businessObject','department','newContract','contractPayment','submissionTime','pid'))
      },
      submitSuccess(formData,flag){
        if(!formData.id){
          let treeData = this.$refs.treeSelect.getCurrTreeData()
          this.expandedRowKeys=[]
          this.getExpandKeysByPid(formData[this.pidField],treeData,treeData)
          this.$emit('ok',formData,this.expandedRowKeys.reverse());
        }else{
          this.$emit('ok',formData,flag);
        }
      },
      getExpandKeysByPid(pid,arr,all){
        if(pid && arr && arr.length>0){
          for(let i=0;i<arr.length;i++){
            if(arr[i].key==pid){
              this.expandedRowKeys.push(arr[i].key)
              this.getExpandKeysByPid(arr[i]['parentId'],all,all)
            }else{
              this.getExpandKeysByPid(pid,arr[i].children,all)
            }
          }
        }
      }
      
      
    }
  }
</script>