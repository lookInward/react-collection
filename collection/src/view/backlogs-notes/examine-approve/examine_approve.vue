<template>
  <el-row style="height:100%;">
    <el-col :span="3" class="examine-aside">
      <p class="aside-title">待办-我的审批</p>
      <el-menu  default-active="0" class="el-menu-vertical-demo">
        <el-menu-item :index="item.code" @click="statusChange(item)" v-for="item in statusList" :key="item.id"> 
          <span slot="title">{{item.name}}
            <span v-if="item.count">{{"("+item.count+")"}}</span>
          </span>
        </el-menu-item>
      </el-menu>

    </el-col>
    <el-col :span="21">
      <div class="content-body">
        <!-- 顶部，包括标题，操作按钮-->
        <div class="bd-top">
          <div class="md clearfix">
            <!-- 1、左边标题 -->
            <div class="md-left">
              <h5>待办-我的审批</h5>
              <!-- <el-button @click="goClerkRole()">切换至文员职位（模拟权限控制）刷新回到主管角色</el-button> -->
            </div>
            <!-- 2、右边操作按钮 -->
            <div class="md-right">
            </div>
          </div>
        </div>
        <div class="bd-main">
          <!-- 切换 -->
          <my-tabs :tabList="tabList" :tabIndex="tabIndex" @changeTab="changeTab">
            <keep-alive>
              <!-- <component :is="currentContent"> </component>  -->
              <!-- 内容-表格 -->
              <el-table ref="multipleTable" height="460" :data="comTb.data" tooltip-effect="dark" show-overflow-tooltip  v-loading="loading" empty-text="暂无数据">
                <el-table-column type="index" width="40">
                </el-table-column>
                <el-table-column label="标题内容">
                  <template slot-scope="scope">
                    {{scope.row.applyContent}}，案件编号
                    <a href="javascript:void(0)" @click="goDetail(scope.row)" style="text-decoration:underline;" class="el-button--text el-button--small">
                      {{scope.row.caseCode}}
                    </a>
                  </template>
                </el-table-column>
                <el-table-column v-for="field in comTb.fields" :align="field.align||'left'" :prop="field.key" :label="field.label" :width="field.width"
                  :key="field.id">
                </el-table-column>
                <el-table-column fixed="right" label="操作" width="180">
                  <template slot-scope="scope">
                    <el-button :type="btn.type" :size="btn.size" v-for="btn in comTb.OperationBtn" :key="btn.id" @click="tbOperate(scope.row,btn)"
                      v-if="scope.row[btn.param]==btn.key" :disabled="btn.disabled">
                      <!-- v-if="scope.row[btn.param]==btn.key" -->
                      {{btn.name}}
                      <!-- {{scope.row.approveStatus!=0?scope.row.approveStatusName:btn.name}} -->
                    </el-button>
                    <span  v-if="comTb.OperationBtn[0].name!='编辑' ">{{scope.row.approveStatus!=0?scope.row.approveStatusName:''}} </span>
                  </template>
                </el-table-column>
              </el-table>
            </keep-alive>
          </my-tabs>
          <pagination :total="total" @changePage="handleCurrentChange" :pageSize="queryParams.pageSize" :currentPage="queryParams.currentPage" @changeSize="changeSize"></pagination>

          <!-- 遮罩 -->
          <el-dialog title="核对欠款金额" :close-on-click-modal="false" :show-close="true" :visible.sync="dialogVisible" :center="true" @close="close">
            <el-form ref='dialogForm' :rules="rules" label-width="100px" :model="dialogForm" :inline="true" size="mini">
              <el-form-item label="案件编号" prop="caseCode">
                <el-input disabled v-model="dialogForm.caseCode" clearable></el-input>
              </el-form-item>
              <el-form-item label="姓名" prop="borrowerName">
                <el-input disabled v-model="dialogForm.borrowerName" clearable></el-input>
              </el-form-item>
              <el-form-item label="身份证号" prop="borrowerIdnumber">
                <el-input disabled v-model="dialogForm.borrowerIdnumber" clearable></el-input>
              </el-form-item>
              <el-form-item label="最新欠款金额" prop="latestDebtMoney">
                <el-input disabled v-model="dialogForm.latestDebtMoney" clearable></el-input>
              </el-form-item>
              <el-form-item label="声明欠款金额" prop="statementAmount">
                <el-input disabled v-model="dialogForm.statementAmount" clearable></el-input>
              </el-form-item>
              <el-form-item label="对账欠款金额" prop="checkAmount">
                <el-input v-model="dialogForm.checkAmount" clearable></el-input>
              </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
              <el-button type="primary" @click="submitCheckAmount(dialogForm)">提 交</el-button>
            </div>
          </el-dialog>

        </div>
      </div>
    </el-col>
  </el-row>

</template>

<script>
  import MyTabs from "../../public-components/my-tabs";
  import pagination from "../../public-components/pagination";
  import qs from "qs";


  const fields = [{
      key: "applyTime",
      label: "提交时间",
      width: "150"
    },
    {
      key: "applicantName",
      label: "申请人",
      width: "80"
    },
    {
      key: "gtasksTypeName",
      label: "类型",
      width: "100"
    }
  ];
  const fields2 = [{
      key: "applyTime",
      label: "提交时间",
      width: "150"
    },
    {
      key: "applicantName",
      label: "申请人",
      width: "80"
    },
    {
      key: "latestDebtMoney",
      label: "最新欠款金额",
      width: "",
      align: 'right'
    },
    {
      key: "statementAmount",
      label: "声明欠款金额",
      width: "",
      align: 'right'
    },
    {
      key: "checkAmount",
      label: "对账欠款金额",
      width: "",
      align: 'right'
    }
  ];
  export default {
    components: {
      MyTabs,
      pagination
    },
    data() {
      return {
        chargeShow:false,
        statusList: [{
          code: '0',
          name: '全部待办',
          count: 0,
        }, {
          code: '1',
          name: '未完成',
          count: 0,
        }, {
          code: '2',
          name: '已完成',
          count: 0,
        }],

        // 切换tabs
        tabIndex: 0,
        tabList: [{
            index: 0,
            name: "全部类型",
            type: 0
          },
          {
            index: 1,
            name: "申请外访",
            type: 1
          },
          {
            index: 2,
            name: "申请函件",
            type: 2
          },
          {
            index: 3,
            name: "申请公安协催",
            type: 3
          },
          {
            index: 5,
            name: "申请留案",
            type: 5
          },
          {
            index: 8,
            name: "申请减免",
            type: 8
          }
        ],
        // 公用表格
        comTb: [],
        // 表格-审批-主管
        tb: {
          data: [],
          fields: fields,
          OperationBtn: [{
              type: 'primary',
              size: 'mini',
              key: '0',
              name: '同意',
              operateAbled: false,
              param: 'approveStatus',
              isShow: false,
              typekey: 1,
              disabled:false,

            },
            {
              type: 'warning',
              size: 'mini',
              key: '0',
              name: '驳回',
              param: 'approveStatus',
              isShow: false,
              typekey: 2,
              disabled:false,

            }
          ]
        },
        // 表格-审批-文员
        tb2: {
          data: [],
          fields: fields2,
          OperationBtn: [{
              type: 'primary',
              size: 'mini',
              key: '0',
              name: '编辑',
              param: 'approveStatus',
              isShow: false,
            },
            {
              type: 'text',
              size: 'mini',
              key: '1',
              name: '已对账',
              param: 'approveStatus',
              isShow: false,

            }
          ]
        },
        loading: false,
        // 分页
        total: 0,
        // 遮罩
        dialogVisible: false,
        dialogForm: {
          caseCode: '',
          borrowerName: '',
          borrowerIdnumber: '',
          latestDebtMoney: '',
          statementAmount: '',
          checkAmount: '',
          id: ''
        },
        rules: {
          checkAmount: [{
              required: true,
              examine: "对账欠款金额不能为空",
              trigger: "blur"
            },
            {
              pattern: /^[1-9][0-9]{0,9}([.][0-9]{0,2})?$/,
              examine: "金额为十位数字且最多两位小数",
              trigger: "blur"
            }
          ],

        },
      };
    },
    computed: {
      queryParams() {
        return {
          currentPage: 1,
          pageSize: 15,
          searchType: 0, //0：全部 1：未完成 2：已完成
          gtasksType: 0 //0:全部 1：外访 2：司法 3：公安
        };
      }
    },
    created() {

      // this.getList(this.queryParams);

      // 权限
      let param = {
        identifier: this.$route.path.slice(1)
      }
      this.$axios.post('/api/assignee/currentUser/getPageResourceByMenu', qs.stringify(param)).then(res => {
        if(res.data.code==0){
            switch (res.data.data[0].identifier) {
                // 主管
                case 'assignee_examine_approve_manager':
                    // this.getNumberList();
                    this.getList(this.queryParams);
                    this.chargeShow=true
                 break;
                // 文员
                case 'assignee_examine_approve_staff':
                 // tab切换的页面资源有改动
                      this.tabList = [{
                        index: 0,
                        name: "申请对账",
                        type: 0
                      }]
                      // 表格里面也有改动
                      this.comTb = this.tb2
                    // this.getNumberList2();
                    this.getList2(this.queryParams);
                     

                break;
            }


        }else{
          this.$util.failCallback(res.data, this);
        }

      }).catch(err => {
        console.log(err);
      });

      // 默认是主管的角色
      this.comTb = this.tb
    },
    methods: {
      // 切换选项卡
      changeTab(tab) {
        this.tabIndex = tab.index;
        this.queryParams.gtasksType = tab.type;
        this.getList(this.queryParams);
        // this.componentIndex=tab.index
        // tab.method()
      },
      // 获取次数-主管
      getNumberList() {
        this.$axios
          .post("/api/assignee/gtasks/getGtasksCountsApprove", {})
          .then(res => {
            console.log(res.data)
            if (res.data.code == 0) {
              console.log(res.data.data)
              this.statusList[0].count = res.data.data.total
              this.statusList[1].count = res.data.data.unfinished
              this.statusList[2].count = res.data.data.finished
            } else {
              this.$util.failCallback(res.data, this);
            }
          })
          .catch(err => {
            this.loading = false;
            console.log(err);
          });
      },
      // 获取次数-文员/assignee/gtasks/getGtasksCountsCheckBill
      getNumberList2() {
        this.$axios
          .post("/api/assignee/gtasks/getGtasksCountsCheckBill", {})
          .then(res => {
            console.log(res.data)
            if (res.data.code == 0) {
              console.log(res.data.data)
              this.totalNumber = res.data.data.total
              this.unfinishedNumber = res.data.data.unfinished
              this.finishedNumber = res.data.data.finished
            } else {
              this.$util.failCallback(res.data, this);
            }
          })
          .catch(err => {
            this.loading = false;
            console.log(err);
          });
      },
      // 审批-主管
      getList(param) {
        //一、 获取标签次数
        this.getNumberList()
        // 二、获取表格列表
        this.loading = true;
        this.$axios
          .post("/api/assignee/gtasks/getGtasksListApprove", param)
          .then(res => {
            this.loading = false;
            // console.log(res.data)
            if (res.data.code == 0) {
              this.tb.data = res.data.data.items;
              this.total = res.data.data.totalNum;
            } else {
              this.$util.failCallback(res.data, this);
            }
          })
          .catch(err => {
            this.loading = false;
            console.log(err);
          });
      },
      // 改变页数
      changeSize(index) {
        this.queryParams.pageSize = index;
        this.getList(this.queryParams)
      },
      // 审批-文员
      getList2(param) {
        //一、 获取标签次数
        this.getNumberList2();
        this.loading = true;
        this.$axios
          .post("/api/assignee/gtasks/getGtasksListCheckBill", param)
          .then(res => {
            this.loading = false;
            if (res.data.code == 0) {
              this.tb2.data = res.data.data.items;
              this.total = res.data.data.totalNum;
            } else {
              this.$util.failCallback(res.data, this);
            }
          })
          .catch(err => {
            this.loading = false;
            console.log(err);
          });
      },
      // 筛选-全部/未完成/已完成
      statusChange(item) {
        this.queryParams.searchType = item.code;
        if(this.chargeShow){
        this.getList(this.queryParams);
        }else{
        this.getList2(this.queryParams);
          
        }
      },
      // 表格-进入详情页
      goDetail(row) {
        console.log(row)

        // url编码
        // let caseCode = this.$util.encrypt(row.caseCode, 'caseDetail');
        // let url = window.location.origin + '/#/worker_case_detail?id=' + caseCode;
        // window.open(url)

        // url编码
        let caseCode = this.$util.encrypt(row.caseCode + '_' + row.caseId.toString() + '_' + row.caseManageId.toString(),
          'caseDetail');
        let url = (window.location.origin ? window.location.origin : '') + '/#/worker_case_detail?id=' + caseCode;
        window.open(url)
      },
      // 表格-操作
      tbOperate(row, btn) {
        // console.log(row)
        if (btn.name == '编辑') {
          // 有遮罩的情况---文员？
          this.dialogVisible = true
          this.dialogForm = row

        } else if (btn.name == '已对账') {
          return false
        } else {
          // 只是点击事件的情况
          btn.disabled = true;
          this.$axios
            .post("/api/assignee/gtasks/disposeGtasks", {
              caseId: row.caseId,
              gtasksType: row.gtasksType,
              id: row.id,
              operatorType: btn.typekey
            })
            .then(res => {
              btn.disabled = false;
              if (res.data.code == 0) {
                this.$message.success(btn.name + '成功')
                this.getList(this.queryParams);

              } else {
                this.$util.failCallback(res.data, this);
              }
            })
            .catch(err => {
              console.log(err);
            });
        }
      },
      // 分页
      handleCurrentChange(index) {
        this.queryParams.currentPage = index;
        this.getList(this.queryParams);
      },
      // 切换至文员角色
      goClerkRole() {
        // tab切换的页面资源有改动
        this.tabList = [{
          index: 0,
          name: "申请对账",
          type: 0
        }]
        // 表格里面也有改动
        this.comTb = this.tb2
        // 获取表格数据
        this.getList2(this.queryParams);

      },
      // 遮罩
      close() {
        this.dialogVisible = false


      },
      // 遮罩-提交
      submitCheckAmount(dialogForm) {
        // console.log(this.$refs.dialogForm)
        this.$refs.dialogForm.validate((valid) => {
          if (!valid) {
            return false
          } else {
            this.$axios.post('/api/assignee/gtasks/disposeGtasksCheckBill', {
                checkAmount: dialogForm.checkAmount,
                id: dialogForm.id
              })
              .then(res => {
                if (res.data.code == 0) {
                  this.close();
                  this.$message({
                    type: 'success',
                    examine: '提交成功'
                  })
                  // 获取文员列表
                  this.getList2(this.queryParams)
                } else {
                  this.$util.failCallback(res.data, this);
                }
              })
              .catch(err => {
                console.log(err);
              })

          }

        })
      }

    }
  };

</script>

<style lang="scss" scoped>
  // .item {
  //   margin-right: 50px;
  // }
  .badge-top {
    margin-bottom: 50px;
    margin-left: 80px;
  }


  .my-tabs {
    margin-top: 30px;
    margin-left: 0 !important;
  }

  .item {
    margin-right: 20px;
  }

  .examine-aside {
    position: relative;
    height: inherit;

    .aside-title {
      height: 70px;
      line-height: 70px;
      position: absolute;
      right: 0;
      left: 0;
      z-index: 9999;
      padding-left: 20px;
      background: #D9DEE4;
      font-weight: bold;

    }
    .el-menu-vertical-demo {
      height: inherit;
      padding-top: 70px;
    }

  }

  .examine-read {
    color: #999;
  }

</style>
