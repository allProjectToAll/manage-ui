<template>
  <div class="my-upload">
    <el-upload
      :action="baseURL+host"
      :drag="true"
      :data="data"
      :with-credentials="true"
      :multiple="multiple"
      :show-file-list="false"
      :on-success="handleSuccess"
      :on-error="handleError"
      :on-progress="handleProgress"
      :on-remove="handleRemove"
      :disabled="disabled||progress>0"
      :headers="headers"
      :name="name"
      :accept="accept"
      :before-upload="beforeUpload">
      <slot name="style">
        <div class="default-style">
          <i class="el-icon-plus"></i>
        </div>
      </slot>
      <span v-loading.body="progress>0" class="loading"></span>
    </el-upload>
  </div>
</template>
<script>
  // 此组件主要处理上传工作
  // 使用示例：
  // <my-upload
  // host="/User/UploadHeadImage"
  // format="image"
  // name="image"
  // :maxSize="2*1024*1024"
  //  @success="handleHeadFileSuccess"
  //   >
  //   <div slot="style" style="width:100%;height:100%;">
  //   </div>
  //   </my-upload>
  export default {
    data() {
      return {
        headers: { Authorization:`Bearer ${this.$store.state.token}`},
        progress: 0,
        baseURL: this.postAjaxBaseUrl
      }
    },
    props: {

      multiple: {
        type: Boolean,
        default: false // 是否支持多选
      },
      format: {
        type: String //文件的格式, image/pdf/excel
      },
      accept: {
        type: String //文件可接受的直接格式，比如 'png'
      },
      minSize:{
        type: Number // 最小尺寸
      },
      maxSize:{
        type: Number // 最大尺寸
      },
      disabled: {
        type: Boolean,
        default: false //是否禁用
      },
      host: {
        type: String, //上传地址
      },
      keyName: {
        type: String | Number, //额外参数
      },
      data: {
        type:Object //附带数据
      },
      name: {
        type: String,
        default:'image' // file 控件
      }
    },
    methods: {
      handleSuccess(data) {
        // 上传成功
        this.progress = 0;
        this.$emit('success', data, this.keyName);
      },
      beforeUpload(file) {
        //验证图片类型和大小
        this.$emit('beforeUpload', file);
        if (this.format == 'image' && file.type.indexOf('image/') == -1) {
          this.$message({ message:this.$t('myUpload.imageFormatErr'),showClose:true, type:'error'});
          return false;
        }
        if (this.format == 'excel' && file.name.indexOf('xls') == -1) {
          this.$message({ message:this.$t('myUpload.excelFormatErr'),showClose:true, type:'error'});
          return false;
        }
        if (this.format == 'pdf' && file.name.indexOf('pdf') == -1) {
          this.$message({ message:this.$t('myUpload.pdfFormatErr'),showClose:true, type:'error'});
          return false;
        }
        if(this.minSize && file.size < this.minSize){
          this.$message({ message:`${this.$t('myUpload.select')} ${this.minSize/1024/1024}${this.$t('myUpload.minSizeTip')}`,showClose:true, type:'error'});
          return false;
        }
        if(this.maxSize && file.size > this.maxSize){
          this.$message({ message:`${this.$t('myUpload.select')} ${this.maxSize/1024/1024}${this.$t('myUpload.maxSizeTip')}`,showClose:true, type:'error'});
          return false;
        }
      },
      handleProgress(progress) {
        // 上传中
        this.progress = progress.percent;
        this.$emit('progress', progress.percent);
      },
      handleError(error) {
        // 出错
        this.$message.error(error);
      },
      handleRemove(){
        console.log('123')
      }
    },
  }
</script>
<style scoped>
  .loading {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 42px;
    height: 42px;
    margin-left: -21px;
    margin-top: -21px;
  }
  .default-style i{
    font-size:50px;
    margin-top:60px;
    color:#b2b2b2;
  }
  .img-style{
    width:100%;
    height:100%;
  }
</style>
