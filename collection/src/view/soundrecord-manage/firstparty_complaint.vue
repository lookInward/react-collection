<template>
  <div>

    <div class="content-body">
      <!-- 顶部，包括标题，操作按钮-->
      <div class="bd-top">
        <div class="md clearfix">
          <!-- 1、左边标题 -->
          <div class="md-left">
            <h5>甲方投诉</h5>
          </div>
          <!-- 2、右边操作按钮 -->
          <div class="md-right">
            <el-button size="mini" type="primary" style="margin:4px 20px 0 0 " @click.prevent.native="exportRecord">导出录音</el-button>
            <!-- <el-button  size="mini" type="primary" @click="openDialog_export(operationBtns[2])">{{operationBtns[2].name}}</el-button> -->
          
          </div>
        </div>
      </div>
      <div class="bd-main">
        <el-form ref="conditionForm" :model="conditionForm" :label-width="this.$util.LABEL_WIDTH"  label-position="right"
          class="condition-form" size="mini">

          <div class="el-col fixed-width">
            <el-form-item label="姓名" prop="borrowerName">
              <el-input v-model="conditionForm.borrowerName" clearable></el-input>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="身份证" prop="borrowerIdnumber">
              <el-input v-model="conditionForm.borrowerIdnumber" clearable></el-input>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="投诉号码" prop="reportPhone">
              <el-input v-model="conditionForm.reportPhone" clearable></el-input>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="案件编号" prop="caseCode">
              <el-input v-model="conditionForm.caseCode" clearable></el-input>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="贷款机构" prop="loanInstitution">
              <el-select clearable v-model="conditionForm.loanInstitution" placeholder="请选择" >
                <el-option v-for="item in dropdownData.loanInstitution" :key="item.id" :label="item.name" :value="item.code"></el-option>
              </el-select>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="批次号" prop="batchCode">
              <el-input v-model="conditionForm.batchCode" placeholder="" clearable></el-input>
            </el-form-item>
          </div>


          <div class="el-col fixed-width">
            <el-form-item label="催收员" prop="collectionId">
              <el-select filterable remote reserve-keyword :remote-method="remoteMethod" clearable v-model="conditionForm.collectionId"
                placeholder="请选择" >
                <el-option v-for="item in collectionId" :key="item.id" :label="item.name" :value="item.code"></el-option>
              </el-select>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="呼叫时间" prop="latestRepayDate">
              <el-date-picker v-model="callTime" value-format="yyyy-MM-dd HH:mm:ss" format="yyyy/MM/dd HH:mm:ss" type="datetimerange" @change="callTimeChange"
                start-placeholder="开始时间" end-placeholder="结束时间" clearable>
              </el-date-picker>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="通话时长" prop="talkTime">
              <el-select clearable v-model="conditionForm.talkTime" placeholder="请选择">
                <el-option v-for="item in dropdownData.talkTime" :key="item.id" :label="item.name" :value="item.code"></el-option>
              </el-select>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="通话人关系" prop="relation">
              <el-select clearable v-model="conditionForm.relation" placeholder="请选择" >
                <el-option v-for="item in dropdownData.relation" :key="item.id" :label="item.name" :value="item.code"></el-option>
              </el-select>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="案件地区" prop="caseArea">
              <el-select clearable v-model="conditionForm.caseArea" placeholder="请选择">
                <el-option v-for="item in dropdownData.caseArea" :key="item.id" :label="item.name" :value="item.code"></el-option>
              </el-select>
            </el-form-item>
          </div>
          <div class="el-col fixed-width">
            <el-form-item label="是否停催" prop="isStop">
              <el-select clearable v-model="conditionForm.isStop" placeholder="请选择">
                <el-option label="All" value=""></el-option>
                <el-option v-for="item in dropdownData.isStop" :key="item.id" :label="item.name" :value="item.code"></el-option>
              </el-select>
            </el-form-item>
          </div>
          <div class="el-col fixed-width form-btns">
            <el-button size="mini" @click="search" type="primary">搜索</el-button>
            <el-button size="mini" @click="resetForm">重置</el-button>
          </div>
        </el-form>
        <el-table ref="multipleTable" height="460" :data="tb.data" tooltip-effect="dark" v-loading="loading" element-loading-text="拼命加载中" element-loading-spinner="el-icon-loading" show-overflow-tooltip  @selection-change="handleSelectionChange">
          <el-table-column type="selection" width="55" fixed="left"></el-table-column>
          <el-table-column v-for="field in tb.fields" :align="field.align||'left'" :prop="field.key" :label="field.label" :width="field.width" :key="field.id">
          </el-table-column>
          <!-- <el-table-column width="80" label="操作" align="left" fixed="right" v-if="operationBtns[0].isShow && operationBtns[1].isShow">
            <template slot-scope="scope">
              <el-button size="mini" v-on:click.stop.prevent="playRecord(scope)" type="text" :class="scope.row.icon" :disabled="scope.row.talkTime == 0 || scope.row.talkTime == null"  v-if="operationBtns[0].isShow">
              </el-button>
              <el-button size="mini" v-on:click.stop.prevent="downloadRecord(scope)" type="text" class="el-icon-download" :disabled="scope.row.talkTime == 0 || scope.row.talkTime == null" v-if="operationBtns[1].isShow">
              </el-button>
            </template>
          </el-table-column> -->
        </el-table>
        <pagination :total="total" :pageSize="conditionForm.pageSize" @changePage="changePage" :current-page="conditionForm.currentPage" @changeSize="changeSize"></pagination>
        <!-- 导出列表 -->
        <!-- <my-export-dialog :dialogData="operationBtns[2].dialog" :conditionData="conditionForm">
        </my-export-dialog> -->
      </div>

    </div>
  </div>
</template>

<script>
  const fields = [{
      key: "recordingId",
      label: '录音编号',
      width: ''
    },
    {
      key: "caseCode",
      label: "案件编号",
      width: "110"
    },
    // {
    //   key: "batchCode",
    //   label: "批次号",
    //   width: ""
    // },
    // {
    //   key: "loanInstitution",
    //   label: "贷款机构",
    //   width: ""
    // },
    {
      key: "complaintName",
      label: "投诉人姓名",
      width: ""
    },
    {
      key: "reportPhone",
      label: "投诉号码",
      width: "100"
    },
    {
      key: "relation",
      label: "关系",
      width: ""
    },

    {
      key: "isStopName",
      label: "是否停催",
      width: ""
    },
    
    {
      key: "complaintReason",
      label: "投诉原因",
      width: "100"
    },
    {
      key: "borrowerName",
      label: "姓名",
      width: ""
    },
    {
      key: "borrowerIdnumber",
      label: "身份证号",
      width: "180"
    },
    {
      key: "caseArea",
      label: "案件地区",
      width: ""
    },
    {
      key: "collectionId",
      label: "催收员（坐席号）",
      width: "120"
    },
    // {
    //   key: "callTime",
    //   label: "呼叫时间",
    //   width: "150"
    // },
    // {
    //   key: "talkTime",
    //   label: "通话时长",
    //   width: ""
    // },
    



  ];
  import qs from "qs";
  import pagination from '../public-components/pagination'
  // import myExportDialog from '../public-components/my-exportDialog.vue'
  
  export default {
    name: "",
    components: {
      pagination,
      // myExportDialog
      
    },
    data() {
      return {
        // 下拉框数据
        dropdownData: {

        },
        collectionId: [],
        // 表格
        // 分页
        // total:0,
        currentPage: 1,
        pageSize: 15,
        // 多选
        // 分发案件遮罩

        rules: {
          borrowerName: [
            this.$util.formRule.userName
          ],
          borrowerIdnumber: [
            this.$util.formRule.ID
          ],
          reportPhone: [
            this.$util.formRule.phone
          ],
          batchCode: [
            this.$util.formRule.batchCode
          ],
          caseCode: [
            this.$util.formRule.caseCode
          ],
        },
        loading: true,
        pickerOptions2: {
          onPick: function (min, max) {}
        },
        conditionForm: {
          batchCode: '',
          borrowerIdnumber: '',
          borrowerName: '',
          //   reportPhone: '',
          callTimeMax: '',
          callTimeMin: '',
          caseArea: '',
          caseCode: '',
          collectionId: '',
          //   department: '',
          loanInstitution: '',
          //   registeredAddress: '',
          talkTime: '',
          reportPhone: '',
          currentPage: 1,
          pageSize: 15
        },
        callTime: [],
        searchForm: {},
        hasSearch: false,
        total: 0,
        operationBtns: [{
            name: "播放",
            identifier: "sponsor_position_search",
            isShow: true
          },
          {
            name: "下载",
            identifier: "sponsor_position_search",
            isShow: true
          },
          {
          name: '导出列表',
          identifier: '',
          dialog: {
            dialogFormVisible: false,
            title: '导出列表',
            submitUrl: '/api/assignee/complaintRecord/export',
            downloadTemplateUrl:'/api/assignee/complaintRecord/downloadTemplate',
            beforeOpenDialogUrl:'/api/assignee/complaintRecord/hasTemplate',
          },
          isShow: true,
         },


        ],
        tb: {
          fields: fields,
          data: [],

        },
        hasSearch: false,
        selected: [],
        audio: null,

      };
    },
    computed: {
      basicParams() {
        return {
          currentPage: this.currentPage,
          pageSize: 15,


        };
      }
    },
    created() {
      if(this.$route.query.caseCode){
        this.conditionForm.caseCode = this.$util.decrypt(this.$route.query.caseCode, 'message');
      }
      // 获取该页的identifier
      let param = {
        identifier: this.$route.path.slice(1)
      };
      // this.$util.getPageResourceByMenu(param,this)
      // 获取下拉列表
      this.getDownList();
      this.searchForm = Object.assign({}, this.conditionForm);
      this.getList(this.conditionForm);
    },
    methods: {
      // 获取下拉列表
      getDownList() {
        this.$axios.post("/api/assignee/complaintRecord/getSearchInfo", {}).then(
            res => {
              if (res.data.code == 0) {
                this.dropdownData = res.data.data;
                this.dropdownData.loanInstitution.unshift({name: 'All',code:''});
                this.dropdownData.talkTime.unshift({name:'All',code:''});
                this.dropdownData.relation.unshift({name:'All',code:''});
                this.dropdownData.caseArea.unshift({name:'All',code:''});
              } else {
                this.$util.failCallback(res.data, this);
              }
            },
            err => {
              console.log(err);
            }
          )
          .catch(err => {
            console.log(err)
          })
      },
      // 承诺还款时间
      callTimeChange(val) {
        if (val) {
          this.conditionForm.callTimeMin = val[0];
          this.conditionForm.callTimeMax = val[1];
        } else {
          this.conditionForm.callTimeMin = '';
          this.conditionForm.callTimeMax = '';
        }
      },

      
      // 播放
      playRecord(scope) {
         //检查是否有audio
        if(this.audio) {
          this.audio.pause()
        }
        let index = scope.row.id;
          if (typeof scope.row.batchCode == 'string') {
          scope.row.batchCode = Number(scope.row.batchCode);
        }else {
          scope.row.batchCode = String(scope.row.batchCode)
        }
         this.tb.data.map( (item) => {
          if(item.id != index) {
            item.icon = 'el-icon-service'
          }
        })
        if (scope.row.icon == 'el-icon-service') {
          scope.row.icon = 'el-icon-loading';
          
          this.$axios({ // 用axios发送post请求
              method: 'post',
              url: '/api/assignee/complaintRecord/download', // 请求地址
              data: {
                id: Number(scope.row.recordingId)
              }, // 参数          
              responseType: 'blob' // 表明服务器返回的数据类型
            })
            .then((res) => { // 处理返回的文件流
              if (res.data.type != 'application/json') {
                const formatArr = (res.headers['content-disposition']).split(".");
                const format = formatArr[formatArr.length - 1];
                const content = res.data;
                const blob = new Blob([content]);
                if ('download' in document.createElement('a')) { // 非IE下载    
                  this.audio = new Audio();
                  this.audio.src = URL.createObjectURL(res.data);
                  if (typeof scope.row.batchCode == 'string') {
                    scope.row.batchCode = Number(scope.row.batchCode)
                  } else {
                    scope.row.batchCode = String(scope.row.batchCode)
                  }
                  scope.row.icon = "el-icon-more";
                  this.audio.play();
                  var self = this;
                  this.audio.onended = function () {
                    if (typeof scope.row.batchCode == 'string') {
                      scope.row.batchCode = Number(scope.row.batchCode);
                    } else {
                      scope.row.batchCode = String(scope.row.batchCode)
                    }
                    self.$message.success('播放完毕');
                    scope.row.icon = "el-icon-service";
                  }
                } else { // IE10+下载
                  navigator.msSaveBlob(blob, fileName)
                }
              } else {
                 this.$message({
                  type: 'error',
                  message: '播放失败'
                })
                if (typeof scope.row.batchCode  == 'string') {
                      scope.row.batchCode = Number(scope.row.batchCode);
                 } else {
                      scope.row.batchCode = String(scope.row.batchCode)
                    }
                scope.row.icon ='el-icon-service'
                
              }
            })
            .catch(err => {
              console.log(err)
            })
        } else if (scope.row.icon == 'el-icon-more') {
          scope.row.icon = 'el-icon-caret-right';
          this.audio.pause();
        } else if (scope.row.icon == 'el-icon-caret-right') {
          scope.row.icon = 'el-icon-more';
          this.audio.play();
        }
      },

      // 下载
      downloadRecord(scope) {
        let id = scope.row.recordingId;
        this.$axios({ // 用axios发送post请求
            method: 'post',
            url: '/api/assignee/complaintRecord/download', // 请求地址
            data: {
              id: Number(scope.row.recordingId)
            }, // 参数
            responseType: 'blob' // 表明服务器返回的数据类型
          })
         .then(res => {
            // if(res.data.code == 0){}
            if (res.data.type != 'application/json') {
              const content = res.data;
              const blob = new Blob([content]);
              const fileName = decodeURI(res.headers["content-disposition"].split("=")[1])
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
                message: '下载失败'
              })
            }
          })
          .catch(err => {
            console.log(err)
          })
      },
      // 导出录音
      exportRecord() {
        if (!this.selected.length) {
          this.$alert("请选择录音记录或者所选无录音文件", "提示", {
            confirmButtonText: "确定",
            type: 'warning'
          });
          return false;
        }
        this.$axios({ // 用axios发送post请求
            method: 'post',
            url: '/api/assignee/complaintRecord/batchExport', // 请求地址
            data: {
              ids: this.selected
            }, // 参数
            responseType: 'blob' // 表明服务器返回的数据类型
          })
          .then(res => {
            // if(res.data.code == 0){}
            if (res.data.type != 'application/json') {
              const content = res.data.data;
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
      // 导出列表
      openDialog_export(btn) {
        btn.dialog.dialogFormVisible = true
      },
      // 选择录音
      handleSelectionChange(arr) {
        this.selected = [];
        arr.forEach(element => {
          if(!element.recordingId) return 
          this.selected.push(element.recordingId)
        });
      },
      // 搜索
      search() {
        this.$refs.conditionForm.validate((valid) => {
          if (valid) {
            this.searchForm = Object.assign({}, this.conditionForm);
            if (this.conditionForm.currentPage == 1) {
              this.getList(this.conditionForm);
            } else {
              this.conditionForm.currentPage = 1;
              this.getList(this.conditionForm);
              this.hasSearch = true;
            }
          }
        })
      },
      // 重置
      resetForm() {
        this.$refs.conditionForm.resetFields();
        this.callTime = []
        this.conditionForm.callTimeMax = '';
        this.conditionForm.callTimeMin = '';
        this.searchForm = Object.assign({}, this.conditionForm)
        if (this.conditionForm.currentPage == 1) {
          this.getList(this.conditionForm);
        } else {
          this.conditionForm.currentPage = 1;
          this.getList(this.conditionForm);
          this.hasSearch = true;
        }
      },
      // 获取表格数据
      getList(data) {
        var queryParams;
        if (data) {
          queryParams = data
        } else {
          queryParams = this.conditionForm
        }
        this.loading = true;
        this.$axios
          .post("/api/assignee/complaintRecord/search", queryParams)
          .then(res => {
            if (res.data.code == 0) {
              this.tb.data = res.data.data.items;
              this.total = res.data.data.totalNum;
              this.tb.data.map((item) => {
                item.icon = "el-icon-service"
              })
            } else {
              this.$util.failCallback(res.data, this);
            }
            this.loading = false;
          })
          .catch(err => {
            console.log(err);
            this.loading = false;
          });
      },
      // 改变页数
      changeSize(index) {
        this.conditionForm.pageSize = index;
        this.searchForm.pageSize = index;
        if(this.conditionForm.currentPage == 1) {
          this.getList(this.conditionForm)
        } else {
          this.conditionForm.currentPage = 1;
          this.getList(this.conditionForm);
        }
      },
      // 表格分页
      changePage(index) {
        this.searchForm.currentPage = index;
        if (this.hasSearch) {
          this.getList(this.conditionForm);
          this.hasSearch = false;
        } else {
          this.conditionForm = Object.assign({}, this.searchForm);
          this.getList(this.searchForm);
        }
      },
      remoteMethod(query) {
        if (query !== '') {
          this.$axios
            .post("/api/assignee/complaintRecord/getStaff", {
              collectionId: query
            })
            .then(res => {
              // console.log(res.data.data);
              if (res.data.code == 0) {
                this.collectionId = res.data.data
              } else {
                this.$util.failCallback(res.data, this);
              }
            })
            .catch(err => {
              console.log(err);
            });

        } else {
          this.collectionId = [];
        }
      }

    }
  };

</script>

<style lang="scss" scoped>

  .el-icon-service,
  .el-icon-download {
    font-size: 18px;
    &:hover {
      color: #67c23a
    }
  }

    .el-icon-service,
  .el-icon-download {
    font-size: 18px;
    &:hover {
      color: #67c23a
    }
  }

  .el-icon-caret-right,
  .el-icon-more,
  .el-icon-loading {
    font-size: 18px;
  }

</style>
