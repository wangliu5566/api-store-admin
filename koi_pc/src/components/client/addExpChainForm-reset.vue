<template>
  <div >
     <div class="add-exp-form">
        <div class="table-page-title add-form">
           <h3>
             知识链
           </h3>
           <span class="yellow-place"></span>
         </div>
        <el-form :model="form" style="width:90%;margin-top:40px;">
        <el-form-item label="知识链名称" :label-width="formLabelWidth">
          <el-input v-model="form.Title" placeholder="请输入知识链名称" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="并列名称" :label-width="formLabelWidth">
          <el-input v-model="form.ScientificName" placeholder="请输入知识链并列名称" type="input" auto-complete="off"></el-input>
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
          <el-select v-model="form.Language" placeholder="请选择知识链编辑时使用的语言">
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

       

        <el-form-item label="知识链描述" :label-width="formLabelWidth">
          <el-input v-model="form.Description" placeholder="请输入知识链描述" :autosize="true" type="textarea" auto-complete="off"></el-input>
        </el-form-item>

         <el-form-item label="参考资料" :label-width="formLabelWidth">
          <el-input v-model="form.Reference" placeholder="请输入参考资料" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="编辑知识链" :label-width="formLabelWidth">
          <el-button size="small" :type="isEditExp?'success':''" @click="openChainDialog">{{isEditExp?'已编辑':'编 辑'}}</el-button>
        </el-form-item> 

        <el-form-item label="修改说明" :label-width="formLabelWidth" v-show="$route.query.isUpdate">
          <el-input v-model="form.updateDescription" placeholder="请输入知识链修改说明" :autosize="true" type="textarea" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>

      <div>
         <el-button type="warning"  style="margin-left: 100px" @click="addNewExplicit">{{$route.query.isUpdate ? '修改并提交':'确定并提交'}}</el-button>
         <el-button @click="backToUpdateOrReview" v-show="$route.query.isUpdate">返 回</el-button>
      </div>

      <el-dialog
        title="创建知识链路"
        :visible.sync="dialogVisible"
        width="60%"
        top="30px"
        >
          <div style="width:90%;margin:30px auto">
            知识列表
          </div>

         <div class="ex-chain">
           <!-- <ul> -->
             <!-- <li v-for="(item,index) in tableData" @click="fullText(item)">{{item.Title}}</li> -->
             <el-button type="primary" plain size="small" :disabled="chainList.length == 0" v-for="(item,index) in tableData" @click="fullText(item)">{{item.Title}}</el-button>
           <!-- </ul> -->
         </div>

         <div style="width:90%;margin:30px auto">
            <el-button size="small" @click="addOneChain">+ 新增链路</el-button>
         </div>
         <ul class="chain-list">
           <li v-for="(item,index) in chainList">
             开始： <el-input v-model="item.start" placeholder="请选择开始" style="width:150px;margin-right:30px" @keydown.native="forbidInput($event)" @focus="inputWillFullText(index,1)"></el-input>
             结束： <el-input v-model="item.end" placeholder="请选择结束" style="width:150px" @keydown.native="forbidInput($event)" @focus="inputWillFullText(index,2)"></el-input>

             <i class="el-icon-remove delete-icon" @click="removeChain(index)"></i>
           </li>
         </ul>


          <div style="width:90%;margin:0 auto;padding:30px 0 2px;border-top:1px solid #ddd">
            <el-button size="small" @click="renderEcharts">生成知识链</el-button>
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
           importances:0,
           multipleSelection: [],
           echartsOptions:null,
           inputBox:{
            row:0,
            which:1,
           }, //用于设置应该填充哪一行哪一个input
           chainList:[
             {
              start:'',
              end:'',
             },
           ],
           chainListKoiNum:[
             {
              FirstKoi:'',
              SecondKoi:'',
              IsStartPoint:false
             }
           ],

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
        forbidInput(e) {
           e.preventDefault();
        },
        setWindow() {
           this.winWidth = window.innerWidth;
           this.winHeight = window.innerHeight;
       },
       //获取KOI列表
       getExplicitWordList() {
          this.$http.get('/KoiNumber/List',{
            params:{
              
            }
          }).then((res) => {
            if (res.data.Code == 200) {
               this.tableData = res.data.Data.ItemList;
               this.tableData.forEach((item,index)=>{
                 item.disabled = false;
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

            apiName = '/ExplicitWordChain/Detail'; 
          }

          this.$http.get(apiName,{
             params:params
          }).then((res) => {
            if (res.data.Code == 200) {
                this.form = res.data.Data.ExplicitWordChain;
                this.form.updateDescription = '';
                //渲染链路
                this.renderExplicitWordChainForUpdate(this.form.Relations);
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
        * [renderExplicitWordChainForUpdate 如果是修改则渲染知识链路]
        * @Author   罗文
        * @DateTime 2017-10-31
        * @param    {[Array]}   relations [链路关系数组]
        * @return   {[type]}             [description]
        */
       renderExplicitWordChainForUpdate(relations) {
          
          this.chainList = [];
          this.chainListKoiNum = [];
          
          relations.forEach((item,index)=>{
            this.chainList.push({
              start:item.FirstKoiName,
              end:item.SecondKoiName
            })

            this.chainListKoiNum.push({
              FirstKoi:item.FirstKoi,
              SecondKoi:item.SecondKoi,
              IsStartPoint:false
            })
          })

       },
      
       /**
        * [openChainDialog 打开模态框，如果是修改，则要渲染echats]
        * @Author   罗文
        * @DateTime 2017-10-31
        * @return   {[type]}   [description]
        */
       openChainDialog() {
          this.dialogVisible = true;
          this.$nextTick(()=>{
            if(this.$route.query.isUpdate) {
               this.renderEcharts();
            }
          })
       },

       addNewExplicit() {
          if(this.validateForm()) {
             this.form.UserId = sessionStorage.userId;

             if(this.form.Relations instanceof Array) {
                this.getSelectKoi();
             }else if(this.chainList[0].start === '') {
                this.$message({
                        message: '请填写知识链！',
                        type: 'error'
                      })
                return;
             }

             //如果是修改
             if(this.$route.query.isUpdate) {
                this.form.KoiNumber = this.$route.query.koiNumber;
             }

             this.$http.post('/ExplicitWordChain/CreateOrUpdate',this.form).then((res) => {
              if (res.data.Code == 200) {
                 this.$message({
                    message: this.$route.query.isUpdate ?'修改知识链成功，已提交审核！':'新增知识链成功，已提交审核！',
                    type: 'success'
                  });

                 let actionType = this.$route.query.isUpdate ? (this.$route.query.id ? 1:2): 1;

                 this.submitApplyLog(res.data.Data,3,actionType);

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
        * [getSelectKoi 获取选中的KOI]
        * @Author   罗文
        * @DateTime 2017-10-27
        * @return   {[type]}   [description]
        */
       getSelectKoi(closeDialog = false) {
          if(this.chainList.length == 0 || this.chainList[0].start === '') {
            this.isEditExp = false;
            this.$message({
                    message: '请填写知识链！',
                    type: 'error'
                  })
            return;
          }

          this.isEditExp = true;

          this.form.Relations = JSON.stringify(this.chainListKoiNum);

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
        * [validateForm 表单验证]
        * @Author   罗文
        * @DateTime 2017-10-25
        * @return   {[Boolean]}   [返回验证通过还是失败]
        */
       validateForm() {
          //验证知识链名
          if(this.form.Title === '' || /[^a-zA-Z0-9_\u4e00-\u9fa5]/g.test(this.form.Title)) {
              this.$message({
                type: 'error',
                message: '知识链名称不能为空，或含有特殊字符！'
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
                message: '知识链描述不能为空，或格式不正确！'
              });
             return false;
          } 


          //验证是否有关联关系
          if(!this.isEditExp) {
              this.$message({
                type: 'error',
                message: '请编辑知识链！'
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
        * [removeChain 移除某一段链路]
        * @Author   罗文
        * @DateTime 2017-11-08
        * @param    {[type]}   index [description]
        * @return   {[type]}         [description]
        */
       removeChain(index) {
         this.chainList.splice(index,1);
         this.chainListKoiNum.splice(index,1);
       },
       
       /**
        * [addOneChain 新增一条链路]
        * @Author   罗文
        * @DateTime 2017-10-30
        */
       addOneChain() {
         this.chainList.push({
          start:'',
          end:''
         });

         this.chainListKoiNum.push({
           FirstKoi:'',
           SecondKoi:'',
           IsStartPoint:false
         });

       },
 
       /**
        * [inputWillFullText 保存即将填入数据的input]
        * @Author   罗文
        * @DateTime 2017-10-30
        * @param    {[Number]}   index [输入框哪一行的索引]
        * @param    {[Number]}     which    [开始还是结束框 1 - 开始 2- 结束]
        * @return   {[type]}         [description]
        */
       inputWillFullText(index,which) {
          //如果是结束框，要对上面知识元列表做限制，只能选择之前已经选择过的知识元
          // if(which == 2) {
          //    if(this.chainList.length > 1 ) {
          //       this.tableData.forEach((pitem,pindex)=>{
          //         pitem.disabled = true;
          //         this.chainList.forEach((citem,cindex)=>{
          //            if(citem.start == pitem.Title || citem.end == pitem.Title) {
          //               pitem.disabled = false;
          //            }
          //         })

          //         this.$set(this.tableData,pindex,pitem);
          //      })
          //    }
          // }else {
          //    this.tableData.forEach((pitem,pindex)=>{
          //       pitem.disabled = false;
          //       this.$set(this.tableData,pindex,pitem);
          //    })
          // }


          this.inputBox = {
            row:index,
            which:which,
            koiNumber:''
          }
       },
       
       /**
        * [fullText 对输入框填充数据]
        * @Author   罗文
        * @DateTime 2017-10-30
        * @param    {[type]}   item [description]
        * @return   {[type]}        [description]
        */
       fullText(item) {
         let which = this.inputBox.which == 1?'start':'end';
         let times = this.inputBox.which == 1?'FirstKoi':'SecondKoi';
         this.$set(this.chainList[this.inputBox.row],which,item.Title);
         
         //当前链路结束知识元不能与开始知识元相同
         if(which == 'end') {
           if(item.Title == this.chainList[this.inputBox.row].start) {
               this.$message({
                  type: 'error',
                  message: '开始和结束不能相同！'
                });

                this.$set(this.chainList[this.inputBox.row],which,'');
                return;
           }
         }

         

         for(let i = 0 ; i < this.chainList.length; i ++) {

            if(this.chainList.length == 1) break;
            //当前链路不能与之前的链路重复
            if(i == this.inputBox.row) continue;

            if((this.chainList[this.inputBox.row].start == this.chainList[i].start && this.chainList[this.inputBox.row].end == this.chainList[i].end) || (this.chainList[this.inputBox.row].start == this.chainList[i].end && this.chainList[this.inputBox.row].end == this.chainList[i].start)) {
               this.$message({
                  type: 'error',
                  message: '不能填写重复的知识链路！'
                });

                this.$set(this.chainList[this.inputBox.row],which,'');
                return;
            }
         }
         

         this.$set(this.chainListKoiNum[this.inputBox.row],times,item.KoiNumber);
       },

       /**
        * [renderEcharts 绘制echarts,知识链图]
        * @Author   罗文
        * @DateTime 2017-10-27
        * @return   {[type]}   [description]
        */
       renderEcharts() {
          //这里需要去重，保留节点唯一
          let hash = [];
          let links = [];

          if(this.chainList.length == 1 && this.chainList[0].start == '') {
             this.isEditExp = false;
          }else this.isEditExp = true;
          
          this.chainList.forEach((item,index)=>{
              if(hash.indexOf(item.start) == -1) {
                 hash.push(item.start);
              }

              if(hash.indexOf(item.end) == -1) {
                 hash.push(item.end);
              }
          })


          if(hash.join('') === '') {
            if(this.myChart) {
               this.isEditExp = false;
               this.myChart.dispose();
            }
            return;
          } 
          
          //渲染节点
          let arr = [];
          let chain = [];
          let max = 0;

          chain = this.ChainCombine(this.chainList);
          
          //找出最大值
          chain.forEach((item,index)=>{
              if(index < chain.length - 1) {
                 if(item.length < chain[index + 1].length) {
                   max = index + 1;
                }
              }

              item.forEach((citem,cindex)=>{
                  //处理关系links
                  links.push({
                    source: citem,
                    target: item[cindex + 1]
                  })
              })
          })

          
          //渲染最长链
          chain[max].forEach((item,index)=>{
              arr.push({
                  name:item,
                  x: 300*index,
                  y: 300
              })
          })


          // [
          //   [1,2,3],
          //   [4,2],
          //   [5,1]
          // ]
          //渲染其他链
          chain.forEach((item,index)=>{
              if(index !== max) {
                 chain[index].forEach((citem,cindex)=>{
                    if(chain[max].indexOf(citem) == -1) {
                       for(var i = 0 ; i < arr.length ; i ++) {
                          if(arr[i].name == citem) break;

                          if(i == arr.length - 1) {
                              arr.push({
                                  name:citem,
                                  x: 300 * (cindex + 1) + 300,
                                  y: 300 * (index + 1) + 300
                              })
                          }
                       }
                       
                    }
                 })
              }
          })



          // hash.forEach((item,index)=>{
          //     //在做位置渲染时，需要先计算出最长的链
          //     arr.push({
          //         name:item,
          //         x: Math.floor(Math.random()*300),
          //         y: Math.floor(Math.random()*300)
          //     })
          // })


          this.echartsOption(arr,links);
          
          if(this.myChart) {
             this.myChart.dispose();
          }

          this.myChart = echarts.init(this.$refs.explicitWordSet);
          this.myChart.setOption(this.echartsOptions);

          this.getSelectKoi(true);
       },

       echartsOption(arr,links) {
         let _this = this;
         this.echartsOptions = {
              title: {
                  text: _this.form.title
              },
              tooltip: {
                trigger: 'item',
                formatter:function(params) {  
                    return params.name;   
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
                      links: [...links],
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

       ChainCombine(chain) {

        // let chain = [ {
        //   start: 'E',
        //   end: 'D'
        // }, {
        //   start: 'A',
        //   end: 'B'
        // },{
        //   start: 'D',
        //   end: 'B'
        // }, {
        //   start: 'B',
        //   end: 'C'
        // }];
        //        alert(chain[0].start);

        let result = new Array();
        for(let i = 0; i < chain.length; i++) {
          let item = new Array();
          item[0] = chain[i].start;
          item[1] = chain[i].end;

          //判断这两个元素是否已经在列表中了

          if(result.length == 0) {
            result.push(item);
          } else {
            var startExist = this.IsExist(result, item[0]);
            var endExist = this.IsExist(result, item[1]);

            if(!startExist && !endExist) {
              //如果两个都不在数据中，则添加到数组
              result.push(item);
              continue;
            }

            //追加到列表后面
            let lastList = this.FindLast(result, item[0]);
            //添加到列表前面
            let firstList = this.FindFirst(result, item[1]);

            if(lastList != null || firstList != null) {
              if(lastList != null) {
                lastList.push(item[1]);
              } else {
                //在第0个位置插入
                firstList.splice(0, 0, item[0]);
              }
            } else {
              //需要处理，end在数据中的情况
              var inList = this.FindIn(result, item[1]);
              if(inList != null) {
                result.push(item);
              }
            }
          }
        }

        return result;
      },

      FindIn(list, value) {
        for(let i = 0; i < list.length; i++) {
          var subList = list[i];
          for(let j = 0; j < subList.length; j++) {
            if(subList[j] == value)
              return subList;
          }
        }
        return null;
      },

      FindFirst(list, value) {
        for(let i = 0; i < list.length; i++) {
          var subList = list[i];
          if(subList[0] == value)
            return subList;
        }
        return null;
      },

      FindLast(list, value) {
        for(let i = 0; i < list.length; i++) {
          var subList = list[i];
          if(subList[subList.length - 1] == value)
            return subList;
        }
        return null;
      },

      IsExist(list, value) {
        for(let i = 0; i < list.length; i++) {
          var subList = list[i];
          for(let j = 0; j < subList.length; j++) {
            if(subList[j] == value)
              return true;
          }
        }
        return false;
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
      width: 90%;
      height: 300px;
      border: 1px solid #337ab7;
    }

    .ex-chain {
      margin: 0 auto;
      width: 90%;
      min-height: 100px;
      max-height: 400px;
      border: 1px solid #337ab7;
      overflow-y: scroll;
      
    }

    .ex-chain button {
        display: flex;
        justify-content: center;
        align-items: center;
        float: left;
        margin: 10px;
        /*border: 1px solid #337ab7;*/
        /*border-radius: 5px;*/
/*        width: 100px;
        height: 50px;
        overflow: hidden;
        text-overflow: ellipsis;
        cursor: pointer;*/

      }

      .ex-chain li:hover {
        background: #337ab7;
        color:white;
      }

      .chain-list {
        width:90%;
        margin:30px auto
      }

      .chain-list li {
        display: flex;
        justify-content: flex-start;
        align-items: center;
        margin: 10px 0
      }

      .delete-icon {
         margin-left: 20px;
         display: inline-block;
         font-size: 22px;
         color:#f7ba2a;
         cursor: pointer;
      }
</style>

