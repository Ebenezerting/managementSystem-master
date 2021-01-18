<template>
    <div class="header">
        <div class="logo">
            <img v-show="!collapse" src="@/assets/img/project.png">
        </div>
        <div :style="{ 'left': !collapse ? '0' : '-160px' }" class="collapse-btn" @click="collapseChage">
            <i v-if="collapse == false" style="color:#838383;" class="el-icon-s-fold"></i>
            <i v-else style="color:#838383;" class="el-icon-s-unfold"></i>
        </div>
        <div class="header-right">
            <div class="header-user-con">
                <!-- 用户头像 -->
                <div class="user-avator1"><img src="../../assets/img/img.jpg"></div>
                <!-- 用户名下拉菜单 -->
                <el-dropdown class="user-name" trigger="click" @command="handleCommand">
                    <span class="el-dropdown-link">
                        {{username}} <i class="el-icon-caret-bottom"></i>
                    </span>
                    <el-dropdown-menu slot="dropdown">
                        <el-dropdown-item divided command="loginout">退出登录</el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
            </div>
        </div>
    </div>
</template>
<script>
    import bus from '../common/bus';
    export default {
        data() {
            return {
                collapse: false,
                name: '',
                msg: 2,
            }
        },
        computed:{
            username(){
                let username = localStorage.getItem('ms_username');
                return username ? username : this.name;
            }
        },
        methods:{
            // 用户名下拉菜单选择事件
            handleCommand(command) {
                if(command == 'loginout'){
                    localStorage.removeItem('ms_username')
                    localStorage.setItem('userArr',[]);
                    this.$router.push('/login');
                }
            },
            // 侧边栏折叠
            collapseChage(){
                this.collapse = !this.collapse;
                bus.$emit('collapse', this.collapse);
            },
            collapseChage2 (){
                bus.$emit('collapse', true);
            }
        },
        mounted(){
            if(document.body.clientWidth < 1500){
                this.collapseChage();
            }
        }
    }
</script>
<style lang="scss">
.header {
    position: relative;
    box-sizing: border-box;
    width: 100%;
    height: 70px;
    font-size: 22px;
    color: #fff;
}
.collapse-btn{
    float: left;
    padding: 0 20px;
    cursor: pointer;
    line-height: 70px;
    color: black;
    position: relative;
    transition: left .3s ease-in-out;
    -moz-transition: left .3s ease-in-out;	/* Firefox 4 */
    -webkit-transition: left .3s ease-in-out;	/* Safari 和 Chrome */
    -o-transition: left .3s ease-in-out;	/* Opera */
}
.header-right{
    float: right;
    padding-right: 8px;
}
.header-user-con{
    display: flex;
    height: 70px;
    align-items: center;
}
.user-name{
    margin-left: 10px;
}
.user-avator1{
    margin-left: 20px;
}
.user-avator1 img{
    display: block;
    width:40px;
    height:40px;
    border-radius: 50%;
}
.el-dropdown-link{
    color: black;
    cursor: pointer;
}
.el-dropdown-menu__item{
    text-align: center;
}
.logo {
    float: left;
    width: 180px;
    /* // height: 70px; */
    background: white;
    padding: 25px 20px 20px 20px;
    transition: width .3s ease-in-out;
    -moz-transition: width .3s ease-in-out;	/* Firefox 4 */
    -webkit-transition: width .3s ease-in-out;	/* Safari 和 Chrome */
    -o-transition: width .3s ease-in-out;	/* Opera */
    img {
        width: 100%;
        height: 100%;
    }
}
</style>
