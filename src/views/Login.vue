<template>
    <div id="main"
         :style="{backgroundImage:'url('+imageurl+')'}">
        <div class="login">
            <h3 class="title">CMDB</h3>
            <hr>
            <el-form ref="form" class="myform" :model="form" label-width="80px">
                <el-form-item label="账号">
                    <el-input v-model="form.username"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input type="password" v-model="form.password"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="onSubmit">登录</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>
<script>
  export default {
    name: "Login",
    data() {
      return {
        form: {
          username: '',
          password: ""
        },
        imageurl: require('E:\\ops-web\\src\\assets\\image\\backimage.jpg')

      }
    },
    methods: {
      onSubmit() {
        this.$http.post('api/v1/user/login/', this.form, {
          transformRequest: [function (data) {
            let params = "";
            for (let index in data) {
              params += index + "=" + data[index] + "&"
            }
            return params
          }]
        }).then(res => {
          if (res.data.code === 0) {
            // 登录成功
            this.$store.commit('clearUser');
            this.$store.commit('clearMenu');
            this.$store.commit("setMenu", res.data.menu);
            this.$store.commit("MenuToRouter", this.$router);
            this.$store.commit("setToken", res.data.token);
            this.$store.commit("setUser", res.data);
            this.$router.push({path: "/"})
          } else {
            this.$message.warning(res.data.msg)
          }
        }).catch(err => {
          console.log(err)
        })
      }
    },
  }
</script>

<style lang="scss" scoped>
    #main {
        background-size: 100% 100%;
        height: 100%;
        position: fixed;
        width: 100%
    }

    .login {
        margin: 250px auto;
        width: 400px;
        border-radius: 10px 10px;
        border: 1px solid #cccccc;
        background: #c0ccda;

        .title {
            font-size: 35px;
            font-weight: bold;
            margin: 10px 10px;
        }

        .myform {
            margin-top: 30px;
            width: 300px;
        }
    }
</style>