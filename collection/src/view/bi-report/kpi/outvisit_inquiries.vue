<template>
  <div class="content-body">
    <!-- 顶部，包括标题，操作按钮-->
    <div class="bd-top">
      <div class="md clearfix">
        <!-- 1、左边标题 -->
        <div class="md-left">
          <h5>催收员外访情况查询</h5>
        </div>
        <!-- 2、右边操作按钮 -->
        <div class="md-right">
          <el-button type="primary" size="mini"  @click="tbexport">导出</el-button>
        </div>
      </div>
    </div>
    <div class="bd-main">
      <el-form ref="conditionForm" :model="conditionForm" :label-width="this.$util.LABEL_WIDTH" label-position="right" class="condition-form"
        size="mini" >
        <div class="el-col fixed-width">
           <el-form-item label="催收部门" prop="departmentId">
              <el-cascader :options="departments" change-on-select v-model="departmentIds" :props="departmentProps" @change="getDepartmentId" >
              </el-cascader>
            </el-form-item>
        </div>
        <div class="el-col fixed-width">
          <el-form-item label="催收员" prop="userId">
              <el-select placeholder="请选择催收员" v-model="conditionForm.userId" clearable>
                <el-option v-for="option in workers" :label="option.name" :value="option.id" :key="option.id"></el-option>
              </el-select>
            </el-form-item>
        </div>
        <div class="el-col fixed-width">
          <el-form-item label="外访时间" prop="visitTime">
            <el-date-picker clearable v-model="conditionForm.visitTime" type="daterange" format="yyyy/MM/dd" range-separator="-" start-placeholder="开始日期"
              end-placeholder="结束日期" value-format="yyyy-MM-dd" @change="getvisitTime">
            </el-date-picker>
          </el-form-item>
        </div>
        <div class="el-col fixed-width form-btns">
          <el-button size="mini" type="primary" @click="search">搜索</el-button>
          <el-button size="mini" @click="reset">重置</el-button>
        </div>
      </el-form>
      <my-table ref="multipleTable" :tb="tb"  :loading="loading" @sortChange="sortChange" @detailDialog="detailDialog"></my-table>
      <el-pagination small @current-change="handleCurrentChange" :current-page.sync="conditionForm.currentPage" :page-size="conditionForm.pageSize"
        layout="sizes,total, prev, pager, next,jumper"  @size-change="changeSize"  :page-sizes="[15, 50, 100]" :total="total" >
      </el-pagination>
        <!-- 详情 -->
    <el-dialog :title="operationBtns[0].dialog.title" :visible.sync="operationBtns[0].dialog.dialogFormVisible" @close="closeDialog(operationBtns[0])" :close-on-click-modal="false">
        <my-table :tb="tb2"  :loading="loading2"></my-table>
        <div class="dialog_submit">
          <el-button size="small"  @click="closeDialog(operationBtns[0])">取消</el-button>
          <el-button size="small" type="primary" @click="tbexport2">导出</el-button>
        </div>
    </el-dialog>
    </div>
    
  </div>

</template>


<script>
  import myTable from '../../public-components/my-table'
  const fields = [
    {
      key: "date",
      label: "日期",
      width: "110",
      type: "0",
      sortable: 'custom'
    },
    {
      key: "userName",
      label: "催收员（催收员ID）",
      width: "",
      type: "0",
    },
    {
      key: "departmentName",
      label: "催收部门",
      width: "",
      type: "0",
    },
    {
      key: "visitTotal",
      label: "外访任务总数量",
      width: "",
      type: "0",
      sortable: 'custom'
      
    },{
      key: "finishTotal",
      label: "已完成外访任务数量",
      width: "",
      type: "0",
      sortable: 'custom'
    },{
      key: "recycleTotal",
      label: "已回收外访任务数量",
      width: "",
      type: "0",
      sortable: 'custom'
    },
  ]
   const fields2 = [
    {
      key: "applyId",
      label: "外访编号",
      width: "",
      type: "0",
    },
    {
      key: "caseId",
      label: "案件编号",
      width: "",
      type: "0",
    },
    {
      key: "beginTimeStr",
      label: "外访开始时间",
      width: "",
      type: "0",
    },
    {
      key: "visitBeginAddress",
      label: "外访起始地址",
      width: "",
      type: "0",
      
    },{
      key: "endTimeStr",
      label: "外访结束时间",
      width: "",
      type: "0",
    },{
      key: "visitEndAddress",
      label: "外访结束地址",
      width: "",
      type: "0",
    },

     {
      key: "duration",
      label: "外访时长（分钟）",
      width: "",
      type: "0",
      
    },{
      key: "uploadTime",
      label: "数据上传时间",
      width: "",
      type: "0",
    },{
      key: "allotTime",
      label: "分配时间",
      width: "",
      type: "0",
    },
  ]
  import qs from 'qs'
  import {
    mapGetters,
    mapMutations
  } from 'vuex'
  export default {
    components: {
      myTable
    },
    data() {
    
      return {
        departments: [],
        departmentIds: [],
        workers: [],
        departmentProps: {
          label: "name",
          value: "id"
        },
        date:'',//查看详情等所要的参数
        userId:'',
        conditionForm: {
          departmentId: "",
          userId:"",
          visitTimeMax:"",
          visitTimeMin:"",
          currentPage: 1,
          pageSize: 15,
           sequence: '',
          sortField: ''
        },
        searchForm: {},
        hasSearch: false,
        loading: true,
        loading2: true,
        
        total: 0,
        operationBtns:[{
          name: '详情',
          identifier: '',
          dialog: {
            dialogFormVisible: false,
            form: {
            },
            rule: {},
            formName: '',
            url: '/api/assignee/visit/statisticDetail',
            title: '详情',
          },
          isShow: true,
        }],
        tb: {
          fields: fields,
          data: [],
          selectionBtn: false,
          height: 520,
          OperationBtn:[{
              type: 'button',
              size: 'mini',
              key: '2',
              name: '详情',
              isShow: true
            },
          ]
        },
        tb2: {
          fields: fields2,
          data: [],
          selectionBtn: false,
          height: 400,
        },
        rules: {
        },
      }

    },
    created() {
      // 获取该页的identifier
      let param = {
        identifier: this.$route.path.slice(1)
      }
      this.searchForm = Object.assign({}, this.conditionForm);
      this.getList(this.conditionForm);
      this.getApartment();
    },
    computed: {
    },
    methods: {

      // 获取还款时间 
      getvisitTime(val) {
        if (!val) {
          this.conditionForm.visitTimeMin = '';
          this.conditionForm.visitTimeMax = '';
        } else {
          this.conditionForm.visitTimeMin = val[0];
          this.conditionForm.visitTimeMax = val[1];
        }
      },

      // 搜索
      search() {
        this.$refs.conditionForm.validate((valid) => {
          if (valid) {
            this.conditionForm.sortField = ""
            this.conditionForm.sequence =""
            this.searchForm = Object.assign({}, this.conditionForm);
            if (this.conditionForm.currentPage == 1) {
              this.getList(this.conditionForm);
            } else {
              this.conditionForm.currentPage = 1;
              this.hasSearch = true;
              this.getList(this.conditionForm);

            }
           this.$refs.multipleTable.cleartbSort();

          }
        })
      },

      // 重置搜索信息
      reset() {
        this.$refs.conditionForm.resetFields();
        this.departmentIds=[]
        this.conditionForm.visitTimeMin = '';
        this.conditionForm.visitTimeMax = '';
        this.conditionForm.sortField = ""
        this.conditionForm.sequence =""
        this.searchForm = Object.assign({}, this.conditionForm)
        if (this.conditionForm.currentPage == 1) {
          this.getList(this.conditionForm);
        } else {
          this.conditionForm.currentPage = 1;
          this.hasSearch = true;
         this.getList(this.conditionForm);

        }
       this.$refs.multipleTable.cleartbSort();

      },

      // 获取列表
      getList(data) {
        var queryParams;
        if (data) {
          queryParams = data
        } else {
          queryParams = this.conditionForm
        }
        this.loading = true;
        this.$axios.post('/api/assignee/visit/statistic', queryParams).then((res) => {
          if (res.data.code == 0) {
            this.tb.data = res.data.data.items;
            this.total = res.data.data.totalNum;
            this.loading = false;
          } else {
            this.$util.failCallback(res.data, this);
            this.loading = false;
          }
        }).catch((err) => {
          console.log(err);
          this.loading = false;
        })
      },
      // 改变页数
      changeSize(index) {
        this.conditionForm.pageSize = index;
        this.searchForm.pageSize = index;
        if(this.conditionForm.currentPage == 1) {
          this.getList(this.conditionForm)
        } else {
          this.conditionForm.currentPage = 1;
        }
      },
      // 获取下拉初始数据
      getApartment() {
        this.$axios.post('/api/assignee/visit/getDepartmentsConditionByBatch',{}).then((res) => {
          if (res.data.code == 0) {
            this.departments = res.data.data;
          } else {
            this.$util.failCallback(res.data, this)
          }
        }).catch((err) => {
          console.log(err)
        })
      },
       // 选择部门
      getDepartmentId(arr) {
        this.conditionForm.departmentId = this.departmentIds[this.departmentIds.length - 1];
        this.conditionForm.userId=""
        this.$axios
          .post("/api/assignee/visit/getStaffsByBatch", {
            departmentId: this.conditionForm.departmentId
          })
          .then(res => {
            if (res.data.code == 0) {
              this.workers = res.data.data;
            } else {
              this.$util.failCallback(res.data, this);
            }
          })
          .catch(err => {
            console.log(err);
          });
      },

      // 跳转分页
      handleCurrentChange(index) {
        this.searchForm.currentPage = index;
        if (this.hasSearch) {
          this.getList(this.conditionForm);
          this.hasSearch = false;
        } else {
          this.searchForm.sequence = this.conditionForm.sequence;
          this.searchForm.sortField = this.conditionForm.sortField;
          this.conditionForm = Object.assign({}, this.searchForm);
          this.getList(this.searchForm);
        }
      },
      // 详情
     detailDialog(scope,btn){
         btn=this.operationBtns[0]
         this.date=scope.row.date
         this.userId=scope.row.userId
         this.operationBtns[0].dialog.dialogFormVisible=true
        this.$axios.post(btn.dialog.url, {
          date: this.date,
          userId:this.userId
        }).then((res) => {
          if (res.data.code == 0) {
            this.tb2.data = res.data.data
            // btn.dialog.dialogFormVisible = true
          } else {
            this.$util.failCallback(res.data, this)
          }
            this.loading2 = false;
          
        }).catch((err) => {
          console.log(err)
            this.loading2 = false;
          
        })

      },
      closeDialog(btn){
          btn.dialog.dialogFormVisible=false
      },
      tbexport(){
        this.$axios({ // 用axios发送post请求
            method: 'post',
            url: '/api/assignee/visit/statistic/export', // 请求地址
            data: {
               visitTimeMin: this.conditionForm.visitTimeMin,
               visitTimeMax: this.conditionForm.visitTimeMax,
               departmentId: this.conditionForm.departmentId,
               userId: this.conditionForm.userId,
            },
            responseType: 'blob' // 表明服务器返回的数据类型
          })
          .then(res => {
            // if(res.data.code == 0){}
            if (res.data.type != 'application/json') {
              const content = res.data;
              const blob = new Blob([content]);
              const fileName = decodeURI(res.headers["content-disposition"].split("=")[1]);
              if ('download' in document.createElement('a')) { // 非IE下载
                const elink = document.createElement('a')
                elink.download = fileName
                elink.style.display = 'none'
                elink.href = URL.createObjectURL(blob)
                document.body.appendChild(elink)
                elink.click()
                URL.revokeObjectURL(elink.href) // 释放URL 对象
                document.body.removeChild(elink)
              } else { // IE10+下载
                navigator.msSaveBlob(blob, fileName)
              }
            } else {
              this.$message({
                type: 'error',
                message: '导出失败'
              })
            }
          })
          .catch(err => {
            console.log(err)
          })

      },
      tbexport2(){
        this.$axios({ // 用axios发送post请求
            method: 'post',
            url: '/api/assignee/visit/statisticDetail/export', // 请求地址
            data: {
               date: this.date,
               userId:this.userId
            },
            responseType: 'blob' // 表明服务器返回的数据类型
          })
          .then(res => {
            // if(res.data.code == 0){}
            if (res.data.type != 'application/json') {
              const content = res.data;
              const blob = new Blob([content]);
              const fileName = decodeURI(res.headers["content-disposition"].split("=")[1]);
              if ('download' in document.createElement('a')) { // 非IE下载
                const elink = document.createElement('a')
                elink.download = fileName
                elink.style.display = 'none'
                elink.href = URL.createObjectURL(blob)
                document.body.appendChild(elink)
                elink.click()
                URL.revokeObjectURL(elink.href) // 释放URL 对象
                document.body.removeChild(elink)
              } else { // IE10+下载
                navigator.msSaveBlob(blob, fileName)
              }
            } else {
              this.$message({
                type: 'error',
                message: '导出失败'
              })
            }
          })
          .catch(err => {
            console.log(err)
          })

      },
      // 表格排序
      sortChange(item) {
        this.conditionForm.sortField = item.prop;
        this.conditionForm.sequence = item.order === 'ascending'?'ASC':'DESC';
        this.getList(this.conditionForm);
      },
    },

   


  }

</script>

<style lang="scss">


</style>
