<template>



  <Layout style="margin-top: 50px;background: #FFF;">
        <Content :style="{margin: '15px 5px 0', }">

          <div v-if="isShowPass">
                <Content style="width: 350px;margin: 120px auto;text-align: center;padding: 40px 10px; border-radius: 10px; box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);">
                  <Form inline @submit.native.prevent>
                    <FormItem prop="password" style="display: inline-block;width: 80%;">
                      <p style="font-size: 24px;"><Icon type="ios-film" size="26" />画廊</p>
                    </FormItem>
                    <FormItem prop="password" style="display: inline-block;width: 80%;">
                      <Input prefix="md-lock" v-model="password" :maxlength="200" size="large" type="password" password   placeholder="画廊访问密码" style="width: 100%;height: 40px;" />
                    </FormItem>
                    <FormItem style="display: inline-block;width: 80%;">
                      <Button type="primary" shape="circle" @click="selectPhoto">&nbsp;<Icon type="md-arrow-round-forward" />&nbsp;</Button>
                    </FormItem>
                  </Form>
                </Content>
          </div>
          <div v-else>
            <viewer class="viewerclass" :images="imglist" :options="options">
              <Row class="animate__animated animate__fadeIn animate__delay-1.5s">
                <Col flex="1" v-for="(item,index) in imglist" :key="index">
                  <div class="imgdivstyle" >
                    <div class="dimback" style="display: inline-block;">
                    <img :class="[item.viewed==true?'imgstyle-max dim':'imgstyle-max']"  class="imgstyle" style="cursor:pointer;" :src="item.imgurl+''"
                         :id="item.id"
                          :alt="item.imgjson"
                    >
                    </div>
                        <div class="img-tool-cover" :style="{bottom:toolBottom+ 'px'}">
                          <Icon style="cursor:pointer;" type="ios-book icostyle" @click.native="imgInfo(item)" title="信息"></Icon>
                        </div>
                  </div>
                </Col>
              </Row>
            </viewer>
            <Page :page-size-opts="pagination.options" v-model="pagination.current" :page-size="pagination.pageSize" :total="pagination.total" @on-change="onCurrentChange" @on-page-size-change="onPageSizeChange" @on-prev="onPrev" @on-next="onNext" show-elevator show-sizer show-total/>
          </div>
          <div :class="viewerBtn">
            <a href="javascript:" id="zoom_in"  @click="clickzoomin"><i class="fa fa-search-plus">&nbsp;放大</i></a>
            <a href="javascript:" id="zoom_out" @click="clickzoomout"><i class="fa fa-search-minus">&nbsp;缩小</i></a>
            <a href="javascript:" id="prev" @click="clickprev"><i class="fa fa-chevron-left">&nbsp;上一张</i></a>
            <a href="javascript:" id="next" @click="clicknext"><i class="fa fa-chevron-right">&nbsp;下一张</i></a>
<!--            <a href="javascript:" id="rotate_left" @click="clickzoomin"><i class="fa fa-rotate-left">&nbsp;左旋转</i></a>-->
<!--            <a href="javascript:" id="rotate_right" @click="clickzoomin"><i class="fa fa-rotate-right">&nbsp;右旋转</i></a>-->
            <a href="javascript:" id="close" @click="clickclose" style="background-color: #FF5722"><i class="fa fa-close">&nbsp;关闭</i></a>
            <a href="javascript:" id="close" @click="clickcopyurl" style="background-color: #FF5722"><i class="fa fa-close">&nbsp;复制链接</i></a>
            <a href="javascript:" id="close" @click="clickopeninnewtab" style="background-color: #FF5722"><i class="fa fa-close">&nbsp;在新标签页打开链接</i></a>
          </div>
        </Content>
        <!--详细信息-->
      </Layout>

</template>

<script>
import {request} from "@/network/request";
// import QRCode from 'qrcodejs2'
import vueQr from 'vue-qr'

export default {
  metaInfo() {
    return {
      title: "画廊|"+this.titlename, // set a title
      meta: [   // set meta
        {
          name: 'keyWords',
          content: this.$store.state.metaInfo.keywords
        },
        {
          name: 'description',
          content: this.$store.state.metaInfo.description
        }
      ],
      link: [{
        rel: 'asstes',
        href: 'http://www.hellohao.cn/'
      }]
    }

  },

  name: "art",
  data () {
    var _this=this;
    return {
      viewerBtn:"not-display",
      isShowPass:false,
      urlKey:null,
      password:null,
      httpText:window.location.protocol,
      hostText:window.location.host,
      viewType:1,//1-大图模式 2-小图
      // QrLog: require("./favicon.ico.bak"),
      upName:null,
      imgage:null,
      isimginfo:false,
      imglist: [],
      pagination: {
        pageNum : 1,
        pageSize : 48,
        total : 100,
        options : [10,20,48,50,100]
      },
      options:{
        viewed (e) {
          console.log(e.type)

          var id = _this.markViewed();
          // var viewer = _this.$el.querySelector('.viewerclass').viewer;
          // var id=viewer.image.id
          console.log(id)
          var img = document.getElementById(id);
          img.classList.add("dim")
          // img.add("dim")
        },
        show(e){
          console.log(e.type)
          _this.viewerBtn = "viewer-btn"
        },
        hide (e) {
          console.log(e.type)
          _this.viewerBtn = "not-display"

        }
      },
      // pageNum:1,
      // pageSize:20,
      selecttype:2,
      type:'picture',
      selectIndex:[],
      bucketname:null,
      submitData:[],
      $stateOBJ:null,
      toolBottom:5,
      nextButloading:false,
      btntext:'加载更多',
      titlename:''
    }
  },

  methods: {
    clickzoomin(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      viewer.zoom(0.1);
    },
    clickzoomout(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      viewer.zoom(-0.1);
    },
    clickprev(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      viewer.prev();
    },
    clicknext(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      viewer.next();
    },
    clickclose(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      viewer.hide();
    },
    clickcopy(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      viewer.hide();
    },
    clickcopyurl(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      var originurl = viewer.image.alt;
      var expains = JSON.parse(originurl).explains
      console.log(expains)
      navigator.clipboard.writeText(expains)
    },
    clickopeninnewtab(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      var originurl = viewer.image.alt;
      var expains = JSON.parse(originurl).explains
      console.log(expains)
      window.open(expains, '_blank').focus()
    },
    picShow: function (e) {
      // v-bind:class="myclass"
      console.log(e.type);
      this.viewerBtn = "viewer-btn"
    },
    onClickImg(){
      this.viewerBtn = "viewer-btn"
    },
    onPageSizeChange(size){
      this.pagination.pageSize = size;
      this.pagination.pageNum = 1;
      this.selectPhoto()
    },
    onCurrentChange(current){
      this.pagination.pageNum = current;
      this.selectPhoto()
    },
    onPrev(current){
      this.pagination.pageNum = current;
      this.selectPhoto()
    },
    onNext(current){
      this.pagination.pageNum = current;
      this.selectPhoto()
    },
    selectPhoto(){
      // this.$Spin.show();
      this.nextButloading=true;
      var paramJson={};
      paramJson.pageNum=this.pagination.pageNum;
      paramJson.pageSize=this.pagination.pageSize;
      paramJson.albumkey = 'TOALBUM'+this.urlKey;
      paramJson.password = this.password;

      request(
          "/getAlbumList",
          paramJson).then(res => {
        console.log(res);
        if(res.status==200){
          var json = res.data;
          if(json.code=='200'){
            this.isShowPass=false;
            var arr = res.data.data.imagesList.rows;
            this.nextButloading=false;
            if(arr.length>0){
              this.imglist=arr;
              // this.imglist=this.imglist.concat(arr);
              // this.pagination.pageNum++;
              var newList=[];
              for (let i=0; i<arr.length; i++){
                var imgInfo={}
                imgInfo.mainImage = arr[i].imgurl;
                imgInfo.explains = arr[i].explains;
                newList[i] = imgInfo
                arr[i].imgjson=JSON.stringify(arr[i])
              }
              console.log(newList)
              this.imglist = arr;
              if(this.imglist.length<this.pagination.pageSize){
                this.btntext='所有数据加载完毕';
              }else{
                this.btntext='加载更多';
              }
            }else{
              this.btntext='所有数据加载完毕';
            }
          }else{
            this.$Message.warning(json.info);
          }
        }else{
          this.$Message.error("请求时出现错误");
        }
        // this.$Spin.hide();
      }).catch(err => {
        // this.$Spin.hide();
        console.log(err);
        this.$Message.error('服务器请求错误');
      })
    },
    countPhoto(){
      // this.$Spin.show();
      this.nextButloading=true;
      var paramJson={};
      paramJson.albumkey = 'TOALBUM'+this.urlKey;

      request(
          "/countAlbum",
          paramJson).then(res => {
        console.log(res);
        if(res.status==200){
          var json = res.data;
          if(json.code=='200'){
            this.isShowPass=false;
            var count = res.data.data.count;
            this.pagination.total = count;
          }else{
            this.$Message.warning(json.info);
          }
        }else{
          this.$Message.error("请求时出现错误");
        }
        // this.$Spin.hide();
      }).catch(err => {
        // this.$Spin.hide();
        console.log(err);
        this.$Message.error('服务器请求错误');
      })
    },
    markViewed(){
      var viewer = this.$el.querySelector('.viewerclass').viewer;
      var originurl = viewer.image.alt;
      var id = JSON.parse(originurl).id
      // this.$Spin.show();
      this.nextButloading=true;
      var paramJson={};
      paramJson.albumkey = 'TOALBUM'+this.urlKey;

      request(
          "/img/markViewed",
          id).then(res => {
        console.log(res);
        if(res.status!=200){
          this.$Message.error("请求时出现错误");
        }
        // this.$Spin.hide();
      }).catch(err => {
        // this.$Spin.hide();
        console.log(err);
        this.$Message.error('服务器请求错误');
      })
      return id
    },
    imgInfo (e) {
      this.isimginfo = true;
      // this.getBucketName(e.source);
      if(e.username!=null){
        this.upName =e.username;
      }else{
        this.upName = '游客';
      }
      this.imgage = e;
    },

    //自动计算大小
    formatBytes(a,b){if(0==a)return"0 Bytes";var c=1024,d=b||2,e=["Bytes","KB","MB","GB","TB","PB","EB","ZB","YB"],f=Math.floor(Math.log(a)/Math.log(c));return parseFloat((a/Math.pow(c,f)).toFixed(d))+" "+e[f]},

    showViewType(){
      this.$Spin.show();
      setTimeout(() =>{
        if(this.viewType==1){
          this.toolBottom = 15;
          this.viewType=2;
          this.selectPhoto(this.$stateOBJ);
        }else{
          this.toolBottom = 5;
          this.viewType=1;
          this.selectPhoto(this.$stateOBJ);
        }
        this.$Spin.hide();
      },1000);

    },
    generateImgInfoJson(image){
      console.log(image)
      var parse = JSON.parse(image);
      return parse
    },
    getAlbum(){
      var param={
        key:'TOALBUM'+this.urlKey
      }
      // request(
      //     "/checkPass",
      //     param).then(res => {
      //   if(res.status==200){
      //     var json = res.data.data;
      //     if(res.data.code=='200'){
      //       if(json.exist){
      //         //有画廊
      //         this.titlename = json.album.albumtitle;
      //         if(json.passType){
      //           this.isShowPass=true;
      //         }else{
                this.isShowPass=false;
                this.countPhoto();
                this.selectPhoto();
      //         }
      //       }else{
      //         // 无画廊
      //         this.$Message.warning("画廊不存在");
      //       }
      //     }else{
      //       this.$Message.error("请求时出现错误");
      //     }
      //   }else{
      //     this.$Message.error("请求时出现错误");
      //   }
      // }).catch(err => {
      //   // this.$Spin.hide();
      //   console.log(err);
      //   this.$Message.error('服务器请求错误');
      // })
    },

    },
  mounted(){
    this.urlKey = window.location.href;
    this.urlKey = window.location.href.substring(this.urlKey.indexOf('h/TOALBUM')+9);
    this.getAlbum();
    //this.selectPhoto();
  },
  components:{
    // BrowseIcon
    // vueQr,
  },
  // metaInfo(){
  //   return {
  //     title: "画廊|"+this.titlename,
  //     meta: [
  //       {
  //         name: "keywords",
  //         content: this.$store.state.metaInfo.keywords
  //       }, {
  //         name: "description",
  //         content: this.$store.state.metaInfo.description
  //       }
  //     ]
  //   }
  // }


}
import { BrowseIcon } from 'tdesign-icons-vue';
</script>

<style>
.dim{
  opacity:0.6; filter: alpha(opacity=60);
}
/*图片样式*/
#picView {
  width: 100%;
  margin: 0 auto;
  font-size: 0;
}

#picView li {
  display: inline-block;
  width: 200px;
  margin-left: 1%;
  padding-top: 1%;
}

#picView li img {
  width: 200px;
}
/*按钮主要样式*/
.viewer-btn {
  bottom: 0;
  left: 0;
  overflow: hidden;
  position: absolute;
  right: 0;
  text-align: center;
  z-index: 10000;
}
.not-display{
  display: none;
}
.viewer-btn a {
  background-color: #1E9FFF;
  border: none;
  color: white;
  padding: 18px 12px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}

.viewer-btn a i {
  font-size: 20px;
}
.example-code-more {
  text-align: center;
  cursor: pointer;
  padding: 30px 0;
  font-size: 16px;
  line-height: 30px;
  font-weight: bold;
}
.imgdivstyle{
  /*width:300px;*/
  height: 160px;
  margin-top: 10px;
  /*background: #e86868;*/
  text-align: center;
  margin-right: 2px;
}
.dimback{
  background: #000;
  width: 300px;
  height: 160px;
  object-fit: cover;
  border-radius:5px;
  border: 1px solid #eee;
  box-shadow: rgba(0, 0, 0, 0.2) 0px 2px 10px 0px;
  transform: translateZ(0);
}
.imgstyle-max{
  width: 300px;
  height: 160px;
  object-fit: cover;
  border-radius:5px;
  border: 1px solid #eee;
  box-shadow: rgba(0, 0, 0, 0.2) 0px 2px 10px 0px;
  transform: translateZ(0);
}
.imgstyle-min{
  width: 155px;
  height: 150px;
  object-fit: cover;
  border-radius:5px;
  border: 1px solid #eee;
  box-shadow: rgba(0, 0, 0, 0.2) 0px 2px 10px 0px;
  transform: translateZ(0);
}
.img-tool-cover{
  display: block;
  position: absolute;
  /*bottom: 5px;*/
  left: 0;
  right: 0;
  background: rgba(255,255,255,.87);
  height: 30px;
  width: 80px;
  box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 6px 0px;
  margin: 0 auto;
  border-radius: 5px;
  text-align: center;
}

.icostyle{
  margin:0 3px 0 3px;
  font-size: 22px;
  line-height: 28px;
  transition: transform 0.2s;
}
.icostyle:hover{
  transform: scale(1.1);
}
.icostylecolor{
  color: #2d8cf0;
}
.img-search{
  position: absolute;
  width: 100%;
  min-width: 512px;
  height: 15px;
  z-index: 1;
}
.img-search-div{
  margin: auto;
  width: 70%;
  height: 70px;
  background: #FFF;
  padding: 13px;
  border-radius: 5px;
  box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 6px 0px;
}
.ivu-btn-icon-only {
  width: 55px;
}
.infotitle{
  color: #464c5b;
  font-size: 14px;
  font-weight: 500;
  margin-right: 10px;
}
.QRCodestyle{
  height: 160px;
  width: 160px;
  position: absolute;
  right: 10px;
  bottom: 10px;
  opacity: 0.7;
}

</style>
