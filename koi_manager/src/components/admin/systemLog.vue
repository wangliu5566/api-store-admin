<template>
  <div class="index-cont" :style="{height:winHeight +'px',width: '92%',margin:'30px auto'}">
    <div class="table-page-title">
       <h3>
         系统日志
       </h3>
       <span class="yellow-place"></span>
     </div>
    <el-row>
        <el-col :span="15" >&nbsp;</el-col>

        <el-col :span="9" >
            <el-input 
              placeholder="请输入用户名查找" 
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
          label="Id"
          width="180">
        </el-table-column> -->
        <el-table-column
          label="操作人"
          width="120"
          align="center"
          >
          <template scope="scope">
             {{scope.row.UserName ? scope.row.UserName : '暂无信息'}}
          </template>
        </el-table-column>
        <el-table-column
          label="操作类型"
          width="120"
          align="center"
          >
          <template scope="scope">
             {{renderActionType(scope.row.ActionType)}}
          </template>
        </el-table-column>
        <el-table-column
          label="操作时间"
          prop="CreateTime"
          width="160"
          align="center">
        </el-table-column>
        <el-table-column
          label="操作内容"
          prop="Title"
          align="center"
          >
        </el-table-column>

        <el-table-column prop="address" label="操作" align="center">
          <template scope="scope">
            <el-button
              size="small"
              type="primary"
              @click="handleKoiDetail(scope.$index,scope.row)">查看详情
            </el-button>

            <el-button
              size="small"
              type="danger"
              v-show="scope.row.ActionType == 4 || scope.row.ActionType == 6"
              @click="handleRejectRes(scope.$index,scope.row)">查看驳回理由
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
          :page-sizes="[15, 20, 30, 40]" 
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
            dialogVisible:false,
            aa:'',
            shenhe:'全部',
            selectType:'全部',
            keyWord:'',
            pageCount: 10,
            currentPage: 1,
            totalCount: 0,
            formData:{},
            itemTitle:'',
            description:'',
            dialogFormVisible:false,
            userName:''
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
       /**
        * [getExplicitWordList 获取管理端列表数据]
        * @Author   罗文
        * @DateTime 2017-10-19
        * @param    {[Boolean]}   isshenhe [传入''，获取所有  true - 已审核  false - 未审核]
        * @return   {[type]}     [description]
        */
       getExplicitWordList() {
          this.$http.get('/History/List',{
            params:{
              ps:this.pageCount,
              cp:this.currentPage,
              UserName:this.keyWord
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


       //切换每页的条数
      handleSizeChange(val) {
        this.pageCount = val;
        this.currentPage = 1;
        this.getExplicitWordList()
      },
      //点击页数，请求第几页
      handleCurrentChange(val) {
        this.currentPage = val;
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
             this.formData = res.data.Data
             this.userName = row.UserName
          } else {
            this.$message({
              message: res.data.Description,
              type: 'error'
            });
          }
        })
      },
 
      /**
       * [handleRejectRes 查看驳回理由]
       * @Author   罗文
       * @DateTime 2017-11-09
       * @param    {[type]}   index [当前行索引]
       * @param    {[type]}   item  [当前行数据]
       * @return   {[type]}         [description]
       */
      handleRejectRes(index,item) {
        console.log(item);
         this.$http.get('/ApplyLog/GetRejectDescription', {
            params: {
              ObjectId:item.ObjectId,
              ObjectType:item.Type,
              ActionType:item.ActionType
            }
          }).then((res) => {
            if(res.data.Code == 200) {
               this.$alert('驳回理由：'+res.data.Data, {
                  confirmButtonText: '确定',
                  center: true,
                  type:'warning'
                });
            } else {
              this.$message({
                message: res.data.Description,
                type: 'error'
              });
            }
          })
         
      }



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

