<template>
  <div >
     <div class="add-exp-form">
        <div class="table-page-title add-form">
           <h3>
             知识簇
           </h3>
           <span class="yellow-place"></span>
         </div>
        <el-form :model="form" style="width:90%;margin-top:40px;">
        <el-form-item label="知识簇名称" :label-width="formLabelWidth">
          <el-input v-model="form.Title" placeholder="请输入知识簇名称" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="并列名称" :label-width="formLabelWidth">
          <el-input v-model="form.ScientificName" placeholder="请输入知识簇并列名称" type="input" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="并列名称语言" :label-width="formLabelWidth">
          <el-select v-model="form.ScientificLanguage" placeholder="请选择并列名称语言">
            <el-option label="汉语" value="汉语"></el-option>
            <el-option label="英语" value="英语"></el-option>
            <el-option label="法语" value="法语"></el-option>
            <el-option label="德语" value="德语"></el-option>
            <el-option label="阿拉伯语" value="阿拉伯语"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="编辑语言" :label-width="formLabelWidth">
          <el-select v-model="form.Language" placeholder="请选择知识簇编辑时使用的语言">
            <el-option label="汉语" value="汉语"></el-option>
            <el-option label="英语" value="英语"></el-option>
            <el-option label="法语" value="法语"></el-option>
            <el-option label="德语" value="德语"></el-option>
            <el-option label="阿拉伯语" value="阿拉伯语"></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="所属分类" :label-width="formLabelWidth">
          <el-select v-model="form.Category" placeholder="请选择所属分类">
            <el-option v-for="(item,index) in cateList" :label="item" :value="item"></el-option>
          </el-select>
        </el-form-item>

        

        <el-form-item label="知识簇描述" :label-width="formLabelWidth">
          <el-input v-model="form.Description" :autosize="true" placeholder="请输入知识簇描述" type="textarea" auto-complete="off"></el-input>
        </el-form-item>


        <el-form-item label="参考资料" :label-width="formLabelWidth">
          <el-input v-model="form.Reference" placeholder="请输入参考资料" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="编辑知识簇" :label-width="formLabelWidth">
          <el-button size="small" :type="isEditExp?'success':''" @click="openChainDialog">{{isEditExp?'已编辑':'编 辑'}}</el-button>
        </el-form-item> 

        <el-form-item label="修改说明" :label-width="formLabelWidth" v-show="$route.query.isUpdate">
          <el-input v-model="form.updateDescription" placeholder="请输入知识簇修改说明" :autosize="true" type="textarea" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>

      <div>
         <el-button type="warning"  style="margin-left: 100px" @click="addNewExplicit">{{$route.query.isUpdate ? '修改并提交':'确定并提交'}}</el-button>
         <el-button @click="backToUpdateOrReview" v-show="$route.query.isUpdate">返 回</el-button>
      </div>

      <el-dialog
        title="创建知识簇"
        :visible.sync="dialogVisible"
        width="60%"
        top="30px"
        >


          <el-table
            ref="multipleTable"
            :data="tableData"
            border
            tooltip-effect="dark"
            style="margin-top:30px"
            @selection-change="handleSelectionChange">
           <el-table-column
              type="selection"
              width="55">
            </el-table-column>
            <el-table-column
              prop="Id"
              label="ID"
              width="180">
            </el-table-column>
            <el-table-column
              prop="Title"
              label="名称">
            </el-table-column>

            <el-table-column
              label="权重"
              align="center"
              >
              <template scope="scope">
                <el-select v-model="weightData[scope.$index]" placeholder="请选择科学名称语言" size="mini" style="width:65px">
                  <el-option label="1" value="1"></el-option>
                  <el-option label="2" value="2"></el-option>
                  <el-option label="3" value="3"></el-option>
                </el-select>
              </template>
            </el-table-column>
          </el-table>

          <div style="margin-top:30px">
            <el-button size="small" @click="renderEcharts">生成知识簇</el-button>
          </div>


          <div ref="explicitWordSet" class="exp-content">
            
          </div>

        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="warning" @click="getSelectKoi(false)">确 定</el-button>
        </span>
      </el-dialog>

    </div>

  </div>
</template>

<script>
   import echarts from 'echarts';
   export default {
     data(){
      return {
           winWidth:window.innerWidth,
           winHeight:window.innerHeight,
           toolsShow:false,
           form: {
              Title: '',
              Category: '哲学',
              Description: '',
              Reference:'',
              Language:'汉语',
              ScientificLanguage:'汉语',
              ScientificName:'',
              KoiNumber:null,
              Relations:'',
              updateDescription:''
           },
           formLabelWidth: '100px',
           dialogVisible:false,
           tableData: [],
           weightData:[],
           selectedExpWeight:[],
           importances:0,
           multipleSelection: [],
           echartsOptions:null,
           hasRender:false,
           isEditExp:false,

           myChart:null,

          cateList:[
           "哲学",
          "经济学",
          "财政学",
          "金融学",
          "经济与贸易",
          "法学",
          "政治学",
          "社会学",
          "民族学",
          "马克思主义理论",
          "公安学",
          "教育学",
          "体育学",
          "中国语言文学",
          "外国语言文学",
          "新闻传播学",
          "历史学",
          "数学",
          "物理学",
          "化学",
          "天文学",
          "地理科学",
          "大气科学",
          "海洋科学",
          "地球物理学",
          "地质学",
          "生物科学",
          "心理学",
          "统计学",
          "力学",
          "机械",
          "仪器",
          "材料",
          "能源动力",
          "电气",
          "电子信息",
          "自动化",
          "计算机",
          "土木",
          "水利",
          "测绘",
          "化工与制药",
          "地质",
          "矿业",
          "纺织",
          "轻工",
          "交通运输",
          "海洋工程",
          "航空航天",
          "兵器",
          "核工程",
          "农业工程",
          "林业工程",
          "环境科学与工程",
          "生物医学工程",
          "食品科学与工程",
          "建筑",
          "安科学与工程",
          "生物工程",
          "公安技术",
          "植物生产",
          "自然保护与环境生态",
          "动物生产",
          "动物医学",
          "林学",
          "水产",
          "草学",
          "基础医学",
          "临床医学",
          "口腔医学",
          "公共卫生与预防医学",
          "中医学",
          "中西医结合","药学","中药学","法医学","医学技术","护理学","管理科学与工程","工商管理","农业经济管理","公管理","图书情报与档案管理","物流管理与工程","工业工程","电子商务","旅游管理","艺术学理论","音乐与舞蹈学","戏剧与影视学","美术学","设计学"],

        }
     },
     methods:{
        setWindow() {
           this.winWidth = window.innerWidth;
           this.winHeight = window.innerHeight;
        },
        //获取KOI列表
        getExplicitWordList() {
          this.weightData = [];

          this.$http.get('/KoiNumber/List',{
            params:{
              
            }
          }).then((res) => {
            if (res.data.Code == 200) {
               this.tableData = res.data.Data.ItemList;
               this.tableData.forEach((item,index)=>{
                 this.weightData.push(1);
               })
            }else {
               this.$message({
                  message: res.data.Description,
                  type: 'error'
                });
            }
          })
        },

       /**
        * [getFormData 修改KOI时获取数据]
        * @Author   罗文
        * @DateTime 2017-10-26
        * @param    {[String]}   data [KOI编码或Id]
        * @param    {[Number]}   type [1 - 通过KOI修改 2- 通过Id修改]
        * @return   {[type]}             [description]
        */
       getFormData(data,type) {
          let params = {};
          let apiName = '';
          if(type == 1) {
            params = {
               KoiNumber:data
             }
            apiName = '/KoiNumber/Detail';
          }else if(type == 2){
            params = {
               Id:data
            }

            apiName = '/ExplicitWordSet/Detail'; 
          }
          this.$http.get(apiName,{
             params:params
          }).then((res) => {
            if (res.data.Code == 200) {
                this.form = res.data.Data.ExplicitWordSet;
                this.form.updateDescription = '';

                this.Relations = res.data.Data.ExplicitWordSet.Relations;
                this.isEditExp = true;
                
            }else {
               this.$message({
                  message: res.data.Description,
                  type: 'error'
                });
             }
          })

       },

        /**
        * [renderExplicitWordChainForUpdate 如果是修改则渲染知识簇]
        * @Author   罗文
        * @DateTime 2017-10-31
        * @param    {[Array]}   relations [知识簇关系集合]
        * @return   {[type]}             [description]
        */
       renderExplicitWordChainForUpdate(relations) {
          if(relations.length === 0) return;

          if(!this.hasRender) {
             let arr = [];
             this.selectedExpWeight = [];

             this.tableData.forEach((item,index)=>{
               relations.forEach((citem,cindex)=>{
                 if(item.KoiNumber == citem.ExplicitWordKoi) {
                    this.selectedExpWeight.push(citem.Importance);
                    arr.push(item);
                 }
               })
             })
            
             this.multipleSelection = arr;
             this.toggleSelection(arr);

             this.hasRender = true;
          } 
       },

       /**
        * [openChainDialog 打开模态框，如果是修改，则要渲染echats]
        * @Author   罗文
        * @DateTime 2017-10-31
        * @return   {[type]}   [description]
        */
       openChainDialog() {
          //渲染知识簇
          this.dialogVisible = true;
          this.$nextTick(()=>{
            if(this.$route.query.isUpdate) {
               this.renderExplicitWordChainForUpdate(this.Relations);
               this.renderEcharts();
            }
          })
       },

      
       
       /**
        * [getSelectKoi 获取选中的KOI]
        * @Author   罗文
        * @DateTime 2017-10-27
        * @param    {[Boolean]}   isUpdate [是否是直接点的修改]
        * @return   {[type]}   [description]
        */
       getSelectKoi(isUpdate = false,closeDialog = false) {

          let arr = [];

          if(isUpdate) {
             this.Relations.forEach((item,index)=>{
                 arr.push({
                    ExplicitWordKoi:item.ExplicitWordKoi,
                    Importance:item.Importance
                 })
              })
          }else {
            this.multipleSelection.forEach((item,index)=>{
               arr.push({
                  ExplicitWordKoi:item.KoiNumber,
                  Importance:this.selectedExpWeight[index]
               })
            })
          }

          this.isEditExp = arr.length === 0 ? false: true;
          this.form.Relations = JSON.stringify(arr);

          this.dialogVisible = closeDialog;
       },

       
       toggleSelection(rows) {
          if (rows) {
            rows.forEach(row => {
              this.$refs.multipleTable.toggleRowSelection(row);
            });
          } else {
            this.$refs.multipleTable.clearSelection();
          }
        },

        handleSelectionChange(val) {
          this.multipleSelection = val;
        },

        /**
         * [addNewExplicit 新增或修改知识簇]
         * @Author   罗文
         * @DateTime 2017-10-31
         */
        addNewExplicit() {
          if(this.validateForm()) {
             this.form.UserId = sessionStorage.userId;
             //如果是修改
             if(this.$route.query.isUpdate) {
                this.form.KoiNumber = this.$route.query.koiNumber;
             }
             
             if(this.form.Relations instanceof Array) {
                this.getSelectKoi(true);
             }

             this.$http.post('/ExplicitWordSet/CreateOrUpdate',this.form).then((res) => {
              if (res.data.Code == 200) {
                 this.$message({
                    message: this.$route.query.isUpdate ?'修改知识簇成功，已提交审核！':'新增知识簇成功，已提交审核！',
                    type: 'success'
                  });

                 let actionType = this.$route.query.isUpdate ? (this.$route.query.id ? 1:2): 1;

                 this.submitApplyLog(res.data.Data,2,actionType);

                 //不管是新增还是修改，都需要提交审核


              }else {
                 this.$message({
                    message: res.data.Description,
                    type: 'error'
                  });
                }
             })
          }
       },

       /**
        * [backToUpdateOrReview 返回操作]
        * @Author   罗文
        * @DateTime 2017-11-09
        * @return   {[type]}   [description]
        */
       backToUpdateOrReview() {
          // this.$route.query.backToPath   1- 修改知识元 2- 簇 3-链  create- 我的申请 update-我的修改
          if(this.$route.query.id) {
            //返回我的申请或我的修改
            this.$router.push({path:'/wrap/updateList',query:{listType:this.$route.query.backToPath}})
            
          }else if(this.$route.query.koiNumber) {
            //返回KOI修改
            if(this.$route.query.backToPath == 'create' || this.$route.query.backToPath == 'update') {
              this.$router.push({path:'/wrap/updateList',query:{listType:this.$route.query.backToPath}});
            }else {
              this.$router.push({path:'/wrap/reViewList',query:{reViewType:this.$route.query.backToPath}});
            }
          }
       },
       
       /**
        * [submitApplyLog 提交审核]
        * @Author   罗文
        * @DateTime 2017-10-26
        * @return   {[type]}   [description]
        */
       submitApplyLog(objectId,objectType,actionType) {
           this.$http.post('/ApplyLog/Create',{
              ObjectId:objectId,
              ObjectType:objectType,
              ActionType:actionType,
              UserId:sessionStorage.userId,
              Description:(this.$route.query.isUpdate && actionType == 1) || actionType == 2 ? this.form.updateDescription :''
           }).then((res) => {
            if (res.data.Code == 200) {
               setTimeout(()=>{
                 let listType = this.$route.query.isUpdate ? 'update':'create';
                 this.$router.push({
                    path:'/wrap/updateList',
                    query:{
                      listType:listType
                    }
                 })
               }, 1000)
            }else {
             this.$message({
                message: res.data.Description,
                type: 'error'
              });
            }
         })
       },
       /**
        * [validateForm 表单验证]
        * @Author   罗文
        * @DateTime 2017-10-25
        * @return   {[Boolean]}   [返回验证通过还是失败]
        */
       validateForm() {
          //验证知识簇名
          if(this.form.Title === '' || /[^a-zA-Z0-9_\u4e00-\u9fa5]/g.test(this.form.Title)) {
              this.$message({
                type: 'error',
                message: '知识簇名称不能为空，或含有特殊字符！'
              });
             return false;
          } 

          //验证参考资料
          /*if(this.form.Reference === '' ||this.form.Reference.indexOf('《') === -1 || this.form.Reference.indexOf('》') === -1) {
              this.$message({
                type: 'error',
                message: '参考资料不能为空，或格式不正确！'
              });
             return false;
          } */

          //验证描述
          if(this.form.Description === '') {
              this.$message({
                type: 'error',
                message: '知识簇描述不能为空，或格式不正确！'
              });
             return false;
          } 


          //验证是否有关联关系
          if(!this.isEditExp) {
              this.$message({
                type: 'error',
                message: '知识集合不能为空，请编辑知识簇！'
              });
             return false;
          } 


          //如果是修改，则还必须验证修改描述
          if(this.$route.query.isUpdate) {
              if(this.form.updateDescription == '') {
                  this.$message({
                    type: 'error',
                    message: '修改描述不能为空！'
                  });
                 return false;
              } 
          }
          

          return true;
       },
       
       /**
        * [setWeight 每次更新数据，设置权重值]
        * @Author   罗文
        * @DateTime 2017-11-09
        */
       setWeight() {
          this.selectedExpWeight = [];
           this.tableData.forEach((item,index)=>{
               this.multipleSelection.forEach((citem,cindex)=>{
                   if(item.Id == citem.Id) {
                      this.selectedExpWeight.push(this.weightData[index]);
                   }
               })
           })
       },

       /**
        * [renderEcharts 绘制echarts,知识簇图]
        * @Author   罗文
        * @DateTime 2017-10-27
        * @return   {[type]}   [description]
        */
       renderEcharts() {
          this.isEditExp = this.multipleSelection.length === 0 ? false:true;


          //这里需要去重，保留节点唯一
          let hash = [];
          let arr = [];
 
          
          this.multipleSelection.forEach((item,index)=>{

              if(hash.indexOf(item.Title) == -1) {
                 hash.push(item.Title);

                 arr.push({
                      name:item.Title,
                      weight:this.selectedExpWeight[index]
                  })
              }
          })


          arr.forEach((item,index)=>{
              item.x = index % 6 * 300;
              item.y = (index - index % 6) / 6 * 300;
          })


          this.echartsOption(arr);

          if(this.myChart) {
             this.myChart.dispose();
          }

          this.myChart = echarts.init(this.$refs.explicitWordSet);
          this.myChart.setOption(this.echartsOptions);


          this.getSelectKoi(false,true);
       },

       echartsOption(arr) {
         let _this = this;
         this.echartsOptions = {
              title: {
                  text: _this.form.title
              },
              tooltip: {
                trigger: 'item',
                formatter:function(params) {  
                    return params.name + ' - 权重值为：'+params.data.weight;   
                }
              },
              animationDurationUpdate: 1500,
              animationEasingUpdate: 'quinticInOut',
              series : [
                  {
                      type: 'graph',
                      layout: 'none',
                      symbolSize: 50,
                      roam: true,
                      label: {
                          normal: {
                              show: true
                          }
                      },
                      edgeSymbol: ['circle', 'arrow'],
                      edgeSymbolSize: [4, 10],
                      edgeLabel: {
                          normal: {
                              textStyle: {
                                  fontSize: 20
                              }
                          }
                      },
                      data: [...arr],
                      // links: [],
                      links: [],
                      lineStyle: {
                          normal: {
                              opacity: 0.9,
                              width: 2,
                              curveness: 0
                          }
                      }
                  }
              ]
          };
       },

     },
     mounted() {
        this.setWindow(); 

        //如果是修改，则获取数据
        if(this.$route.query.isUpdate) {
            if(this.$route.query.koiNumber) {
               this.getFormData(this.$route.query.koiNumber,1);
            }else if(this.$route.query.id) {
               this.getFormData(this.$route.query.id,2);
            }else {
               this.$message({
                  type: 'error',
                  message: '未获取到修改数据！'
                });
            }
        }

        this.getExplicitWordList();

        window.onresize = ()=>{
           this.setWindow();
        }
     },

     watch:{
      'multipleSelection':{
        handler:function(nv) {
           this.setWeight();
        },
        deep:true
      },

      'weightData':{
        handler:function(nv) {
           this.setWeight();
        },
        deep:true
      },


     }
   }
</script>
<style lang="css" scoped>
    .add-exp-form {
      margin: 0 auto;
      width: 90%;
    }


    .exp-content {
      margin: 30px auto;
      overflow: hidden;
      width: 90%;
      height: 300px;
      border: 1px solid #337ab7;
    }
</style>

