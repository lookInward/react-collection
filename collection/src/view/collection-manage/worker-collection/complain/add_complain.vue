<template>
  <div class="form">
    <el-dialog :close-on-click-modal="false" :show-close="true" :visible.sync="complainVisible"  :center="true"  @close="close('complainForm')" title="添加投诉">
    <el-form ref='complainForm' :rules="rules" label-width="100px" :model="complainForm" >
      <el-form-item label="投诉人姓名：" prop="complaintName" class="item">
        <el-input v-model="complainForm.complaintName"  size="mini" clearable></el-input>
      </el-form-item>
      <el-form-item label="投诉号码：" prop="reportPhone" class="item">
        <el-input v-model="complainForm.reportPhone" size="mini" clearable></el-input>
      </el-form-item>
      <el-form-item label="投诉内容：" prop="">
        <el-input  v-model="complainForm.complaintReason" placeholder="请输入投诉内容" type="textarea" :rows="6" :maxlength="200" size="mini" clearable></el-input>
      </el-form-item>
      <div  class="dialog-footer footer">
        <el-button type="primary" @click="submitComplain('complainForm')" size="mini">提 交</el-button>
        <el-button type="primary" @click="reset('complainForm')" size="mini">取消</el-button>
      </div>
    </el-form>
    </el-dialog> 
  </div>
</template>

<script>

export default {
  name: "complain",
  props: {
    complainVisible: {
      type: Boolean,
    },
    onLineData: {
      type: Object
    },
  },
  data() {
    return {
      complainForm: {
        caseId:this.onLineData.caseId,
        complaintName : '',
        reportPhone: '',
        complaintReason: ''
      },
      rules: {
        complaintName: [
          this.$util.formRule.userName
        ],
        reportPhone: [
            this.$util.formRule.phone
          ],
        textarea: {

        }  
      }
    }
  },
  computed: {
  },
  created() {
  },
  methods: {
    // 遮罩
    submitComplain(){
        this.$axios
        .post("/api/assignee/complaintPrewarning/add",this.complainForm)
        .then(res => {
          if (res.data.code == 0) {
            this.$message.success("添加投诉预警成功")
            this.$emit('complainClose');
            this.reset(complainForm)
            
          } else {
            this.$util.failCallback(res.data, this);
          }
        })
        .catch(err => {
          console.log(err);
        });
    },
    reset(complainForm){
        this.$refs.complainForm.resetFields();
        this.$emit('complainClose');
    }, 
    close(complainForm){
        this.$emit('complainClose');
        this.reset(complainForm)

    },

  }
};
</script>
<style scoped lang="scss">
  .el-dialog .el-form {
    width: 70%;
    margin: 0 auto;
    .el-form-item {
      display: block
    }
    .el-select {
      width: 100%;
    }
    .el-input_innner,.el-textarea_inner{
      width:100%;
    }
    .el-date-editor  {
      width: 100%;
    }
    .footer {
        text-align: left;
        margin-left:100px;
    }

  }
</style>

