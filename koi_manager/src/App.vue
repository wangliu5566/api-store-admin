<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>

export default {
  name: 'app',
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color:#444;
  height: 100%;
  width: 100%;
  overflow-x: hidden;
}

::-webkit-scrollbar{width:0px;}
::-webkit-scrollbar-track{background-color:#bee1eb;}
::-webkit-scrollbar-thumb{background-color:#00aff0;}

.title {
   margin-top :20px;
   color: #3372b7;
}

.yellow-place {
      display: inline-block;
      margin-bottom: 10px;
      height: 5px;
      width: 26px;
      background: #fec708;
    }

 .table-page-title h3{
 	font-weight: normal;
 	margin-bottom: 15px;
 }   

 .add-form {
 	margin-top: 30px;
 }


.background-img {
  position: fixed;
}

.left-top {
  left: 20.6%;
  /*top:4.3%;*/
}

.right-bottom {
  right: 0;
  bottom:0;
}


 .content-title {
   position: fixed;
   z-index: 8;
   right: 0;
   top:0;
   display: flex;
   justify-content: flex-end;
   align-items: center;
   height: 40px;
   width: 100%;
   background: white;
   border-bottom: 1px solid #eee;
   box-shadow: 0px 0px 10px #999;
   -webkit-app-region: drag;
 }

 .content-title i {
   display: inline-block;
   line-height: 40px;
   text-align: center;
   /*width: 40px;*/
   height: 40px;
   padding: 0 15px;
 }

 .content-title i:hover {
   color:#0393e8;
   cursor: pointer;
   /*background: #2b579a;*/
 }

  .pageExchange {
   text-align: right;
   margin-top:10px;
   position:relative;
   z-index:5
 }




</style>
