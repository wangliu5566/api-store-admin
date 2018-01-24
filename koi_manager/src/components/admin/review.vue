<template>
  <div class="index-cont" :style="{width: '92%',margin:'30px auto'}">
    <div class="table-page-title">
       <h3>
         查询
       </h3>
       <span class="yellow-place"></span>
     </div>

    <el-row>
      <el-col :span="3">&nbsp;</el-col>
      <el-col :span="14">
        <el-select v-model="expType" @change="getExplicitWordList"  placeholder="选择KOI类型" style="width:130px">
          <el-option label="全部" value="0"></el-option>
          <el-option label="知识元" value="1"></el-option>
          <el-option label="知识簇" value="2"></el-option>
          <el-option label="知识链" value="3"></el-option>
        </el-select> 

        <el-select v-model="firstShenhe" @change="getExplicitWordList"  placeholder="一审状态" style="width:130px">
          <el-option label="全部" value="3"></el-option>
          <el-option label="正在审核" value="0"></el-option>
          <el-option label="审核通过" value="1"></el-option>
          <el-option label="驳回" value="2"></el-option>
        </el-select>

        <el-select v-model="secondShenhe" @change="getExplicitWordList"  placeholder="二审状态" style="width:130px">
          <el-option label="全部" value="3"></el-option>
          <el-option label="正在审核" value="0"></el-option>
          <el-option label="审核通过" value="1"></el-option>
          <el-option label="驳回" value="2"></el-option>
        </el-select>
      </el-col>
      <el-col :span="1">&nbsp;</el-col>
      <el-col :span="6">
        <el-input 
          placeholder="请输入名称" 
          v-model="keyWord" 
          @keyup.enter.native="getExplicitWordList"
          >
          <el-button slot="append" icon="el-icon-search" @click="getExplicitWordList"></el-button>
        </el-input>
      </el-col>
    </el-row>    

    <el-table
      :data="tableData"
      :border="true"
      style="margin-top:30px">
<!--         <el-table-column
          prop="Id"
          label="ID"
          width="180">
        </el-table-column> -->
        <el-table-column
          prop="Title"
          label="名称"
          >
        </el-table-column>
        <el-table-column
          label="类型"
          >
          <template scope="scope">
             {{scope.row.ObjectType ? (scope.row.ObjectType == 1 ? '知识元':(scope.row.ObjectType == 2 ? '知识簇' :'知识链')): '暂无信息' }}
          </template>
        </el-table-column>
        <el-table-column prop="address" label="一审状态" align="center">
        <template scope="scope">
          {{scope.row.FirstAuditState == 0 ? '正在审核...':(scope.row.FirstAuditState == 1 ? '审核通过':'') }} 
          <el-button size="small" type="danger" v-if="scope.row.FirstAuditState == 2" @click="showUnAcceptDescription(scope.row.FirstAuditDescription)" >已驳回</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="secondAuditState" label="二审状态" align="center">
        <template scope="scope">
          {{scope.row.FirstAuditState == 2?'——':(scope.row.SecondAuditState == 0 ? '正在审核...':(scope.row.SecondAuditState == 1 ? '审核通过': ''))}} 
          <el-button size="small" type="danger" v-if="scope.row.FirstAuditState !== 2 && scope.row.SecondAuditState == 2" @click="showUnAcceptDescription(scope.row.SecondAuditDescription)" >已驳回</el-button>
        </template>
      </el-table-column>


        <el-table-column
          prop="KoiNumber"
          label="KOI编码">
          <template scope="scope">
              {{scope.row.KoiNumber ? scope.row.KoiNumber : '未通过审核'}}
          </template>
        </el-table-column>

        <el-table-column prop="address" label="操作" align="center">
          <template scope="scope">
            <el-button
              size="small"
              type="primary"
              @click="handleKoiDetail(scope.$index,scope.row)">查看详情
            </el-button>
          </template>
        </el-table-column>
    </el-table>


    <!-- 分页 -->
     <div class="block pageExchange">
        <el-pagination 
          @size-change="handleSizeChange" 
          @current-change="handleCurrentChange" 
          :current-page="currentPage" 
          :page-sizes="[10, 20, 30, 40]" 
          :page-size="pageCount" 
          layout="total, sizes, prev, pager, next" 
          :total="totalCount">
       </el-pagination>
     </div>


    <el-dialog width="60%" :title="itemTitle" :center="true" top="30px" :visible.sync="dialogFormVisible">
      <koi-detail :formData="formData" :description="description" :userName="userName"></koi-detail>

      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">关 闭</el-button>
      </div>
    </el-dialog> 
  </div>
</template>

<script>
   import KoiDetail from './koiDetail.vue';
   export default {
     data(){
      return {
           winWidth:window.innerWidth,
           winHeight:window.innerHeight,
           toolsShow:false,
            tableData: [],
            tableData1: [],
            tableData2: [],
            dialogVisible:false,
            aa:'',
            selectType:'全部',
            keyWord:'',
            pageCount: 10,
            currentPage: 1,
            totalCount: 0,
            formData:{},
            itemTitle:'',
            description:'',
            userName:'',
            dialogFormVisible:false,
            firstShenhe: '',
            secondShenhe: '',
            expType: '',
        }
     },
     components:{
        'koi-detail':KoiDetail,
       },
     methods:{
       setWindow() {
           this.winWidth = window.innerWidth;
           this.winHeight = window.innerHeight;
       },
       showUnAcceptDescription(description){        
        this.$alert('驳回理由：'+description, {
              confirmButtonText: '确定',
              center: true,
              type:'warning'
            });
       },


       /**
        * [getExplicitWordList 获取管理端列表数据]
        * @Author   罗文
        * @DateTime 2017-10-19
        * @return   {[type]}     [description]
        */
       getExplicitWordList() {
          let firstShenhe = this.firstShenhe || 3;
          let secondShenhe = this.secondShenhe || 3;
          let expType = this.expType || 0;

          this.$http.get('/ApplyLog/List',{
            params:{
               ps:this.pageCount,
               cp:this.currentPage,
               Title:this.keyWord,
               ObjectType:expType,
               SecondAuditState:secondShenhe,
               FirstAuditState:firstShenhe,
            }
          }).then((res) => {
            if (res.data.Code == 200) {
               this.tableData = res.data.Data.ItemList;
               console.log(this.tableData)
               this.totalCount = res.data.Data.RecordCount;
            }else {
               this.$message({
                  message: res.data.Description,
                  type: 'error'
               });
            }
          })
       },
       handleOpen(key, keyPath) {
        console.log(typeof key);

       },
       handleClose(key, keyPath) {
        console.log(key, keyPath);
       },
       handleEdit(index, row) {
         this.$confirm('审核通过?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$message({
              type: 'success',
              message: '审核通过(功能演示)！'
            });
          }).catch(() => {
        
          });
       },
       handleDelete(index, row) {
          this.$confirm('确认驳回?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$message({
              type: 'success',
              message: '已驳回(功能演示)！'
            });
          }).catch(() => {
        
          });
       },
      handleClose(done) {
        this.$confirm('确认关闭？')
          .then(_ => {
            done();
          })
          .catch(_ => {});
      },

      //切换每页的条数
      handleSizeChange(val) {
        this.pageCount = val;
        this.currentPage = 1;
        this.getExplicitWordList()
      },
      //点击页数，请求第几页
      handleCurrentChange(val) {
        this.currentPage = val;
        console.log('in')
        this.getExplicitWordList()
      },


      /**
       * [handleKoiDetail 查看操作详情]
       * @Author   罗文
       * @DateTime 2017-11-08
       * @param    {[type]}   index [description]
       * @param    {[type]}   row   [description]
       * @return   {[type]}         [description]
       */
      handleKoiDetail(index,row) {
        this.dialogFormVisible = true;
        this.itemTitle = row.Title;

        let apiName = '/ExplicitWord/Detail';

        switch(row.ObjectType) {
          case 1:
            apiName = '/ExplicitWord/Detail';
            break;
          case 2:
            apiName = '/ExplicitWordSet/Detail';
            break;
          case 3:
            apiName = '/ExplicitWordChain/Detail';
            break;
          default:
            // statements_def
            break;
        }

        this.$http.get(apiName, {
          params: {
            Id:row.ObjectId
          }
        }).then((res) => {
          if(res.data.Code == 200) {
             this.formData = res.data.Data;
             this.description = row.Description;
             this.userName = row.UserName;
          } else {
            this.$message({
              message: res.data.Description,
              type: 'error'
            });
          }
        })
      },
     },
     mounted() {
        this.setWindow();
        this.getExplicitWordList();
        window.onresize = ()=>{
           this.setWindow();
        }
     }
   }
</script>
<style lang="css">
   .side-bar {
      background: rgb(238,241,236);
   }

   .el-table__body-wrapper {
      overflow: hidden !important;
   }
</style>

