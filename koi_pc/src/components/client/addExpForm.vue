<template>
  <div :style="{height:winHeight +'px'}" >

     <div class="add-exp-form">
        <div class="table-page-title add-form">
           <h3>
             知识元
           </h3>
           <span class="yellow-place"></span>
         </div>

        <el-form :model="form" style="width:90%;margin-top:40px;">
        <el-form-item label="知识元名称" :label-width="formLabelWidth">
          <el-input v-model="form.Title" placeholder="请输入知识元名称" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="并列名称" :label-width="formLabelWidth">
          <el-input v-model="form.ScientificName" placeholder="请输入并列名称" type="input" auto-complete="off"></el-input>
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
          <el-select v-model="form.Language" placeholder="请选择知识元编辑时使用的语言">
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

        

        <el-form-item label="知识元描述" :label-width="formLabelWidth">
          <el-input v-model="form.Description" placeholder="请输入知识元描述" :autosize="true" type="textarea" auto-complete="off"></el-input>
        </el-form-item>



        <el-form-item label="参考资料" :label-width="formLabelWidth">
          <el-input v-model="form.Reference" placeholder="请输入参考资料" auto-complete="off"></el-input>
        </el-form-item>


        <el-form-item label="修改信息" :label-width="formLabelWidth" v-show="$route.query.isUpdate">
          <el-input v-model="form.updateDescription" placeholder="请输入知识元修改描述" :autosize="true" type="textarea" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>

      <div>
         <el-button type="warning"  style="margin-left: 100px" @click="addNewExplicit">{{$route.query.isUpdate ? '修改并提交':'确定并提交'}}</el-button>
         <el-button @click="backToUpdateOrReview" v-show="$route.query.isUpdate">返 回</el-button>
      </div>
    </div>

  </div>
</template>

<script>
   export default {
     data(){
      return {

           winWidth:window.innerWidth,
           winHeight:window.innerHeight,
           toolsShow:false,
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
           form: {
              Title: '',
              Category: '哲学',
              Description: '',
              Reference:'',
              Language:'汉语',
              ScientificLanguage:'汉语',
              ScientificName:'',
              KoiNumber:null,
              updateDescription:''
           },
           formLabelWidth: '100px'

        }
     },
     methods:{
        setWindow() {
           this.winWidth = window.innerWidth;
           this.winHeight = window.innerHeight;
       },
       /**
        * [getFormData 修改KOI时获取数据]
        * @Author   罗文
        * @DateTime 2017-10-26
        * @param    {[String]}   koiNumber [KOI编码]
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

            apiName = '/ExplicitWord/Detail'; 
          }

          this.$http.get(apiName,{
             params:params
          }).then((res) => {
            if (res.data.Code == 200) {
                this.form = res.data.Data.ExplicitWord;
                this.form.updateDescription = '';
            }else {
               this.$message({
                  message: res.data.Description,
                  type: 'error'
                });
             }
          })
       },

       addNewExplicit() {
          if(this.validateForm()) {
             this.form.UserId = sessionStorage.userId;
             //如果是修改
             if(this.$route.query.isUpdate) {
                this.form.KoiNumber = this.$route.query.koiNumber;
             }

             console.log(this.form)

             this.$http.post('/ExplicitWord/CreateOrUpdate',this.form).then((res) => {
              if (res.data.Code == 200) {
                 this.$message({
                    message: this.$route.query.isUpdate ?'修改知识元成功，已提交审核！':'新增知识元成功，已提交审核！',
                    type: 'success'
                  });

                 let actionType = this.$route.query.isUpdate ? (this.$route.query.id ? 1:2): 1;

                 this.submitApplyLog(res.data.Data,1,actionType);

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
              Description: (this.$route.query.isUpdate && actionType == 1) || actionType == 2 ? this.form.updateDescription :''
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
          //验证知识元名
          if(this.form.Title === '' || /[^a-zA-Z0-9_\u4e00-\u9fa5]/g.test(this.form.Title)) {
              this.$message({
                type: 'error',
                message: '知识元名称不能为空，或含有特殊字符！'
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
                message: '知识元描述不能为空，或格式不正确！'
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
       }

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
</style>

