<template>
  <div>
    <el-dialog title="号码分配" :close-on-click-modal="false" width="40%" :show-close="true" :visible.sync="dialogVisible" :center="true"
      @close="close">
      <el-form ref="conditionForm" label-position="right" label-width="100px" class="dialog-main" size="small">
         <el-form-item label="选择部门" prop="aDepartmentIds" clearable >
            <el-cascader :options="departments" v-model="aDepartmentIds"  @change="handleChange" change-on-select :props="departmentProps" size="small"> </el-cascader>
        </el-form-item>
        <el-form-item label="催收员：" prop="userId">
          <el-select v-model="userId" size="small" clearable>
            <el-option v-for="item in this.dropDownData" :label="item.name" :value="item.id" :key="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
        </el-form-item>
      </el-form>
      <div class="dialog_submit">
          <el-button size="small" @click="reset">取消</el-button>
          <el-button size="small" type="primary" @click="submit">提交</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    name: 'telAllotDialog',
    props: {
      row: {
        type: Object
      },
      dialogVisible: {
        type: Boolean
      }
    },
    data() {
      return {
        save: false,
        userId: '',
        dropDownData: [],
        departments: [],
        departmentId: '',
        departmentProps: {
          label: "name",
          value: "id"
       },
      }
    },

    methods: {
      // 关闭
      close() {
        this.$emit('dialogClose');
        this.userId = ''
      },

      //保存
      submit() {
        if(!this.userId) {
          this.$alert('未选择催收员?', '提示', {
          confirmButtonText: '确定',
          type: 'warning'
         })
         return false;
        }
        this.$axios
          .post("/api/assignee/seatAllot/allot", {
            id: this.row.id,
            userId: this.userId
          })
          .then(res => {
            if (res.data.code == 0) {
              this.$message({
                type: 'success',
                message: '保存成功'
              })
              this.save = true;
              this.$emit('dialogClose', this.save)
            } else {
              this.$util.failCallback(res.data, this);
            }
          })
          .catch(err => {
            console.log(err);
          });

      },

      // 重置
      reset() {
        this.close();
      },
      // 获取催收员列表
      getCollectionId() {
        this.$axios
          .post("/api/assignee/seatAllot/getStaffs", {departmentId:this.departmentId})
          .then(res => {
            if (res.data.code == 0) {
              this.dropDownData = res.data.data;
            } else {
              this.$util.failCallback(res.data, this);
            }
          })

      },
      // 获取部门列表
      getApartment() {
        this.$axios
          .post("/api/assignee/seatAllot/getDepartmentsCondition", {})
          .then(res => {
            if (res.data.code == 0) {
              this.departments = res.data.data;
            } else {
              this.$util.failCallback(res.data, this);
            }
          })
          .catch(err => {
            console.log(err);
          });
      },
       // 选择部门
      handleChange(data) {
        this.userId = '';
        this.departmentId = this.aDepartmentIds[this.aDepartmentIds.length-1];
        this.getCollectionId()
      },
    },
    created() {
      console.log(this.row)
     // this.getCollectionId();
      this.getApartment();
    }
  }

</script>
<style lang="scss" scoped>
  .el-form-item {
    position: relative;
  }

  .form-btns {
    position: relative;
    top: -20px;
    padding-bottom: 40px;

  }

</style>
