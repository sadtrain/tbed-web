<template>
    <Card class="animate__animated animate__fadeIn" style="width:390px;margin: 80px auto;min-width: 40vh;">
      <div style="text-align:center">
        <Form ref="formInline" :model="formInline" :rules="ruleInline" inline>

          <Layout style="background: #fff;">
            <Header style="color: #515a6e">
              <Icon type="md-finger-print" style="font-size: xx-large;" />
              <span style="font-size: large; font-weight: bold;">Register</span>
            </Header>
            <Content>
            <Form ref="formInline" :model="formInline" :rules="ruleInline" inline @submit.native.prevent>
              <FormItem prop="username" style="display: inline-block;width: 80%;">
                <Input prefix="md-contact" size="large" :maxlength="20" show-word-limit v-model="formInline.username" autocomplete="new-password" placeholder="用户名 可数字字母下划线" style="width: 100%;height: 40px;" />
              </FormItem>
              <FormItem prop="email" style="display: inline-block;width: 80%;">
                <Input prefix="md-mail" size="large" v-model="formInline.email" :maxlength="200" placeholder="邮箱" autocomplete="new-password" style="width: 100%;height: 40px;" />
              </FormItem>
              <FormItem prop="password" style="display: inline-block;width: 80%;">
                <Input prefix="md-lock" type="password" size="large"  :maxlength="200"   v-model="formInline.password" autocomplete="new-password" placeholder="用户密码" style="width: 100%;height: 40px;" />
              </FormItem>
              <FormItem prop="password" style="display: inline-block;width: 80%;">
                <Input prefix="md-lock" type="password" size="large"  :maxlength="200"   v-model="formInline.passwordtow" autocomplete="new-password" placeholder="确认用户密码" style="width: 100%;height: 40px;" />
              </FormItem>
              <FormItem style="display: inline-block;width: 80%;">
              <Row>
                <Col span="16">
                  <Input prefix="md-barcode" placeholder="验证码" :maxlength="10" :rules="ruleInline" autocomplete="new-password"  v-model="formInline.verifyCode" size="large"   style="width: 100%;height: 40px;" />
                </Col>
                <Col span="4" offset="1">
                  <img @click="reloadCode" :src="verifyCodeURL" />
                </Col>
              </Row>
              </FormItem>
              <FormItem>
                <ButtonGroup shape="circle">
                  <Button type="primary" @click="handleSubmit('formInline')">注 册</Button>
                </ButtonGroup>
              </FormItem>
            </Form>
            </Content>
          </Layout>

        </Form>
      </div>
    </Card>


</template>
<script>

import {request} from "../network/request"
export default {
  metaInfo() {
    return {
      title: '注册|'+this.$store.state.metaInfo.webname+this.$store.state.metaInfo.splitline+this.$store.state.metaInfo.websubtitle, // set a title
      meta: [   // set meta
        {
          name: 'keyWords',
          content: this.$store.state.metaInfo.keywords
        },
        {
          name: 'description',
          content: this.$store.state.metaInfo.description
        }
      ]
    }
  },

  data () {
    return {
      verifyCodeURL:null,
      userToken:null,
      formInline: {
        username:'',
        email: '',
        password: '',
        passwordtow: '',
        verifyCode:''//verifyCodeForRegister
      },
      ruleInline: {
        username: [
          { required: true, message: ' ', trigger: 'blur' },
          { type: 'string', min: 5,max:20, message: '用户名长度为5-20位数字或字母', trigger: 'blur' }
        ],
        email: [
          { required: true,min: 5,max:200, message: ' ', trigger: 'blur' }
        ],
        password: [
          { required: true, message: ' ', trigger: 'blur' },
          { type: 'string', min: 5,max:200, message: '密码长度最短不得小于5位', trigger: 'blur' }
        ],
        passwordtow: [
          { required: true, message: ' ', trigger: 'blur' },
          { type: 'string', min: 5,max:200, message: '密码长度最短不得小于5位', trigger: 'blur' }
        ],
        verifyCode:[
          { required: true, message: ' ', trigger: 'blur' }
        ]
      }
    }
  },
  mounted() {
    // console.log("组件传送开始"+this.$serverHost)
    this.$emit('getRouterInfo', "ahahha");
    this.getVerifyCode();
  },
  methods: {
    handleSubmit(name) {
      this.$refs[name].validate((valid) => {
        if (valid) {
          if(!this.formInline.username){
            this.$Message.info('请输入用户名');
            return false;
          }
          if(!this.formInline.email){
            this.$Message.info('请输入邮箱');
            return false;
          }
          if(!this.formInline.password){
            this.$Message.info('请输入密码');
            return false;
          }
          if(!this.formInline.passwordtow){
            this.$Message.info('请再次输入密码');
            return false;
          }
          if(!this.formInline.verifyCode){
            this.$Message.info('请输入验证码');
            return false;
          }
          if(this.formInline.password != this.formInline.passwordtow){
            this.$Message.info('密码输入不一致');
            return false;
          }
          // console.log(JSON.stringify(this.formInline));
          // var params = {
          //   data: this.formInline
          // };
          this.$Spin.show();
          request(
              "/user/register",
              this.formInline).then(res => {
                this.$Spin.hide();
                if(res.status==200){
                  var json = res.data;
                  if(json.code=='200'){
                    // this.$Message.success();
                    this.$Notice.success({
                      title: json.info,
                      // desc: nodesc ? '' : 'Here is the notification description. Here is the notification description. '
                    });
                    this.$router.replace("/login");//index
                  }else{
                    this.reloadCode();
                    this.formInline.verifyCode = '';
                    this.$Message.error(json.info);
                  }
                }else{
                  this.$Message.error("请求时出现错误");
                }
          }).catch(err => {
            this.$Spin.hide();
            console.log(err);
            this.reloadCode();
            this.$Message.error('服务器请求错误');
          })
        } else {
          this.$Message.error('别尼玛瞎输入!');
        }
      })
    },
    login(){
      this.$Spin.show();
      // this.$Spin.hide();
    },
    reloadCode(){
      this.getVerifyCode()

    },
    //获取验证码
    getVerifyCode(){
      request(
          "/verifyCodeForRegister",
          {}).then(res => {
        if(res.status==200){
          var json = res.data;
          localStorage.setItem('verifyCodeForRegister',json.data.codeKey)
          this.verifyCodeURL = 'data:image/gif;base64,'+json.data.codeImg

        }else{
          this.$Message.error("请求时出现错误");
        }
      }).catch(err => {
        console.log(err);
        this.$Message.error('服务器请求错误');
      })
    },
  }

}
</script>


<style scoped>
.ivu-form-item-error .ivu-input {
  border: 2px solid #e86868;
}
.ivu-form-item-error .ivu-input-group-prepend {
  background-color: #fff;
  border: 2px solid #e86868;
}
.ivu-layout-header {
  background: #fff;
}

.ivu-card-bordered {
  border: 0;
  border-color: #e8eaec;
  box-shadow: 0 2px 5px 1px rgba(64,60,67,.16);
  /*box-shadow: 0 0 10px #ebebec;*/
}


</style>
