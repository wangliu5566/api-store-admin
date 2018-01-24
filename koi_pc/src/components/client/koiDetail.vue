<template>
  <div >
     <div class="add-exp-form">
        <el-form :model="form" style="width:95%;">
          <el-form-item label="知识类型" :label-width="formLabelWidth">
            <el-input v-model="ExpType" placeholder="请输入知识类型" auto-complete="off" :disabled="true"></el-input>
          </el-form-item>

          <el-form-item label="名称" :label-width="formLabelWidth">
            <el-input v-model="form.Title" placeholder="请输入名称" auto-complete="off" :disabled="true"></el-input>
          </el-form-item>

          <el-form-item label="并列名称" :label-width="formLabelWidth">
            <el-input v-model="form.ScientificName" placeholder="请输入并列名称" type="input" auto-complete="off" :disabled="true"></el-input>
          </el-form-item>

          <el-form-item label="并列名称语言" :label-width="formLabelWidth">
            <el-select v-model="form.ScientificLanguage" placeholder="请选择并列名称语言"  :disabled="true">
              <el-option label="汉语" value="汉语"></el-option>
              <el-option label="英语" value="英语"></el-option>
              <el-option label="法语" value="法语"></el-option>
              <el-option label="德语" value="德语"></el-option>
              <el-option label="阿拉伯语" value="阿拉伯语"></el-option>
            </el-select>
          </el-form-item> 


          <el-form-item label="编辑语言" :label-width="formLabelWidth">
            <el-select v-model="form.Language" placeholder="请选择编辑时使用的语言"  :disabled="true">
              <el-option label="汉语" value="汉语"></el-option>
              <el-option label="英语" value="英语"></el-option>
              <el-option label="法语" value="法语"></el-option>
              <el-option label="德语" value="德语"></el-option>
              <el-option label="阿拉伯语" value="阿拉伯语"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="所属分类" :label-width="formLabelWidth">
            <el-select v-model="form.Category" placeholder="请选择活动区域" :disabled="true">
              <el-option label="科学" value="科学"></el-option>
              <el-option label="建筑" value="建筑"></el-option>
              <el-option label="历史" value="历史"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="描述" :label-width="formLabelWidth">
            <el-input v-model="form.Description" placeholder="请输入描述" :autosize="true" type="textarea" auto-complete="off" :disabled="true"></el-input>
          </el-form-item>

          

          <el-form-item label="参考资料" :label-width="formLabelWidth">
            <el-input v-model="form.Reference" placeholder="请输入参考资料" auto-complete="off" :disabled="true"></el-input>
          </el-form-item>

           <el-form-item label="知识关系图" :label-width="formLabelWidth" v-if="ExpTypeNum !== 1">
             <div ref="explicitWordSet" class="exp-content">
              
             </div>
           </el-form-item>


           <el-form-item label="日志列表" :label-width="formLabelWidth">
              <el-card class="box-card">
                <div v-for="item in logList" :key="item.Id" class="text item">
                  {{item.CreateTime + '     ' +item.Title}}
                </div>
              </el-card>
           </el-form-item>
        </el-form>

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
              Title: '钢筋混泥土加固',
              Category: '科学',
              Reference: '《钢筋混泥土设计手册》、《钢筋混泥土预设基础》',
              Description:'钢筋混凝土（英文：Reinforced Concrete或Ferroconcrete），工程上常被简称为钢筋砼（tong）。是指通过在混凝土中加入钢筋钢筋网、钢板或纤维而构成的一种组合材料与之共同工作来改善混凝土力学性质的一种组合材料。为加劲混凝土最常见的一种形式。[1] ',
              Language:'汉语',
              ScientificLanguage:'汉语',
              ScientificName:''
           },
           ExpType:'知识元',
           ExpTypeNum:1,
           formLabelWidth: '100px',
           isShowSearch:false,
           keyWords:'',
           logList:[]

        }
     },
     props:['formData'],
     methods:{
        setWindow() {
           this.winWidth = window.innerWidth;
           this.winHeight = window.innerHeight;
       },
       
       /**
        * [getLogData 查看详情时获取这个知识的操作日志]
        * @Author   罗文
        * @DateTime 2017-11-08
        * @return   {[type]}   [description]
        */
       getLogData(objectId,objectType) {
           if(!objectId || !objectType) return;

           this.$http.get('/Message/ObjectList',{
              params:{
                  ObjectId:objectId,
                  ObjectType:objectType,
               }
           }).then((res) => {
            if (res.data.Code == 200) {
                this.logList = res.data.Data.ItemList;
            }else {
             this.$message({
                message: res.data.Description,
                type: 'error'
              });
            }
         })
       },
       
       /**
        * [renderForm 渲染搜索结果]
        * @Author   罗文
        * @DateTime 2017-10-26
        * @param    {[Object]}   data [表单数据]
        * @return   {[type]}        [description]
        */
       renderForm(data) {
           if(data.ExplicitWord) {
              this.renderByExplicitWord(data);
              this.ExpType = '知识元';
              this.ExpTypeNum = 1;
           }else if(data.ExplicitWordSet) {
              this.ExpType = '知识簇';
              this.ExpTypeNum = 2;
              this.renderByExplicitWordSet(data);
           }else if(data.ExplicitWordChain) {
              this.ExpType = '知识链';
              this.ExpTypeNum = 3;
              this.renderByExplicitWordChain(data);
           }

           this.getLogData(data.ObjectId,data.ObjectType);
       },

       /**
        * [renderByExplicitWord 渲染知识元表单]
        * @Author   罗文
        * @DateTime 2017-10-26
        * @param    {[Object]}   data [表单数据]
        * @return   {[type]}        [description]
        */
       renderByExplicitWord(data) {
          this.form = data.ExplicitWord;
       },

       /**
        * [renderByExplicitWordSet 渲染知识簇表单]
        * @Author   罗文
        * @DateTime 2017-10-26
        * @param    {[Object]}   data [表单数据]
        * @return   {[type]}        [description]
        */
       renderByExplicitWordSet(data) {
          this.form = data.ExplicitWordSet;
          this.renderEcharts(data.ExplicitWordSet.Relations,2);
       },

       /**
        * [renderByExplicitWordChain 渲染知识链表单]
        * @Author   罗文
        * @DateTime 2017-10-26
        * @param    {[Object]}   data [表单数据]
        * @return   {[type]}        [description]
        */
       renderByExplicitWordChain(data) {
          this.form = data.ExplicitWordChain;
          this.renderEcharts(data.ExplicitWordChain.Relations,3);
       },

       /**
        * [renderEcharts 绘制echarts,知识簇图]
        * @Author   罗文
        * @DateTime 2017-10-27
        * @param    {[Array]}   data [知识元关系数组]
        * @param    {[Number]}   type [2 - 知识簇 3-知识链]
        * @return   {[type]}   [description]
        */
       renderEcharts(data,type) {
          if( data.length === 0) return;

          let arr = [];
          let links = [];
          //这里需要去重，保留节点唯一
          let hash = [];


          if(type == 2) {
            //组织知识簇的数据
            data.forEach((item,index)=>{
              if(hash.indexOf(item.ExplicitWordKoiName) == -1) {
                 hash.push(item.ExplicitWordKoiName);
                 arr.push({
                      name:item.ExplicitWordKoiName,
                      weight:item.Importance
                 })
              }
            })

            arr.forEach((item,index)=>{
                item.x = index % 6 * 300;
                item.y = (index - index % 6) / 6 * 300;
            })
            
          }else if(type == 3) {

            // data.forEach((item,index)=>{
            //   links.push({
            //     source: item.FirstKoiName,
            //     target: item.SecondKoiName
            //   })

            //   if(hash.indexOf(item.FirstKoiName) == -1) {
            //      hash.push(item.FirstKoiName);
            //      arr.push({
            //         name:item.FirstKoiName,
            //         x: Math.floor(Math.random()*300),
            //         y: Math.floor(Math.random()*300)
            //      })
            //    }

            //    if(hash.indexOf(item.SecondKoiName) == -1) {
            //      hash.push(item.SecondKoiName);
            //      arr.push({
            //         name:item.SecondKoiName,
            //         x: Math.floor(Math.random()*300),
            //         y: Math.floor(Math.random()*300)
            //      })
            //    }
            // })



            //渲染节点
            // let arr = [];
            let chain = [];
            let max = 0;

            chain = this.ChainCombine(data);
            
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
          } 


          this.echartsOption(arr,links);

          this.$nextTick(()=>{
             let myChart = echarts.init(this.$refs.explicitWordSet);
             myChart.setOption(this.echartsOptions);
          })

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
          item[0] = chain[i].FirstKoiName;
          item[1] = chain[i].SecondKoiName;

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
        window.onresize = ()=>{
           this.setWindow();
        }

        this.renderForm(this.formData);
     },

     watch:{
       'formData':function(nv) {
          console.log(this.formData)
          this.renderForm(this.formData);
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
      overflow: hidden;
      border: 1px solid #337ab7;
    }


    .text {
      font-size: 14px;
      color:#bebebe;
    }


    .box-card {
      width: 100%;
    }
</style>

