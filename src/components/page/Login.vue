<template>
    <div class="containerLogin">
        <div class="top">
            <div class="headerLogin">
                <a href="/">
                    <img src="~@/assets/img/project.png" class="logoGin" alt="logo">
                    <!-- <span class="title">超管后台</span> -->
                </a>
            </div>
        </div>
        <div class="main">
            <el-form :model="param" :rules="rules" ref="login" label-width="0px" class="ms-content">
                <el-form-item prop="username">
                    <el-input v-model="param.username" placeholder="username">
                        <el-button slot="prepend" icon="el-icon-lx-people"></el-button>
                    </el-input>
                </el-form-item>
                <el-form-item prop="password">
                    <el-input
                        type="password"
                        placeholder="password"
                        v-model="param.password"
                        @keyup.enter.native="submitForm()"
                    >
                        <el-button slot="prepend" icon="el-icon-lx-lock"></el-button>
                    </el-input>
                    <!-- <img :src="param.url" alt=""> -->
                </el-form-item>
                <div class="login-btn">
                    <el-button type="primary" @click="submitForm()">登录</el-button>
                </div>
                <!-- <p class="login-tips">Tips : 用户名和密码随便填。</p> -->
            </el-form>
        </div>
        <div class="footer">
        <div class="links">
          <a href="_self">帮助</a>
          <a href="_self">隐私</a>
          <a href="_self">条款</a>
        </div>
        <div class="copyright">
          Copyright &copy; 2018 vueComponent
        </div>
      </div>
    </div>
</template>

<script>
export default {
    data: function() {
        return {
            param: {
                username: 'admin',
                password: '123123',
            },
            rules: {
                username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
                password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
            },
        };
    },
    methods: {
        submitForm() {
            this.$refs.login.validate(valid => {
                if (valid) {
                    this.$message.success('登录成功');
                    localStorage.setItem('ms_username', this.param.username);
                    sessionStorage.setItem('tagsList', JSON.stringify([]));
                    this.$router.push('/');
                } else {
                    this.$message.error('请输入账号和密码');
                    console.log('error submit!!');
                    return false;
                }
            });
        },
    },
};
</script>

<style lang="scss" scoped>
.containerLogin {
        width: 100%;
        min-height: 100%;
        background: #f0f2f5 url(~@/assets/background.svg) no-repeat 50%;
        background-size: 100%;
        padding: 250px 0 144px;
        position: relative;
    a {
        text-decoration: none;
    }

    .top {
        text-align: center;

        .headerLogin {
            height: 44px;
            line-height: 44px;
            margin-bottom: 30px;
            .badge {
                position: absolute;
                display: inline-block;
                line-height: 1;
                vertical-align: middle;
                margin-left: -12px;
                margin-top: -10px;
                opacity: 0.8;
            }

            .logoGin {
                // height: 44px;
                vertical-align: top;
                // margin-right: 16px;
                border-style: none;
            }

            .title {
                font-size: 33px;
                color: rgba(0, 0, 0, .85);
                font-family: Avenir, 'Helvetica Neue', Arial, Helvetica, sans-serif;
                font-weight: 600;
                position: relative;
                top: 2px;
            }
        }

    }
    .main {
        min-width: 260px;
        width: 368px;
        margin: 0 auto;
    }

    .footer {
        position: absolute;
        width: 100%;
        bottom: 0;
        padding: 0 16px;
        margin: 48px 0 24px;
        text-align: center;

        .links {
            margin-bottom: 8px;
            font-size: 14px;
            a {
                color: rgba(0, 0, 0, 0.45);
                transition: all 0.3s;
                &:not(:last-child) {
                    margin-right: 40px;
                }
            }
        }
        .copyright {
            color: rgba(0, 0, 0, 0.45);
            font-size: 14px;
        }
    }
    .login-button {
        font-size: 16px;
        height: 40px;
        width: 100%;
    }
    .user-login-other {
        text-align: left;
        margin-top: 24px;
        line-height: 22px;

        .item-icon {
            font-size: 24px;
            color: rgba(0, 0, 0, 0.2);
            margin-left: 16px;
            vertical-align: middle;
            cursor: pointer;
            transition: color 0.3s;
            &:hover {
                color: #1890ff;
            }
        }

        .register {
            float: right;
        }
    }
    .login-btn {
        text-align: center;
    }
    .login-btn button {
        width: 100%;
        height: 36px;
        margin-bottom: 10px;
    }
    .login-tips {
        text-align: right;
        font-size: 12px;
        line-height: 30px;
    }
}
</style>