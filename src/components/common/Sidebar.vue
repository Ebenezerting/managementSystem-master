<template>
    <div :style="{ 'top': !collapse ? '70px' : '0' }" class="sidebar">
        <el-menu class="sidebar-el-menu" :default-active="onRoutes" :collapse="collapse" background-color="#191a23"
            text-color="rgb(191, 203, 217)" active-text-color="#fff" unique-opened router>
            <template v-for="item in items">
                <template v-if="item.subs">
                    <el-submenu :index="item.index" :key="item.index">
                        <template slot="title">
                            <i id="icontent" :class="item.icon"></i>
                            <span slot="title">{{ item.title }}</span>
                        </template>
                        <template v-for="subItem in item.subs">
                            <!-- <el-submenu v-if="subItem.subs" :index="subItem.index" :key="subItem.index">
                                <template slot="title">{{ subItem.title }}</template>
                                <el-menu-item v-for="(threeItem,i) in subItem.subs" :key="threeItem.index" :index="threeItem.index">
                                    {{ threeItem.title }}
                                </el-menu-item>
                            </el-submenu> -->
                            <div class="aser">
                                <el-menu-item class="entr" :index="subItem.index" :key="subItem.index">
                                    <!-- <i :class="subItem.icon"></i> -->
                                    <span>{{ subItem.title }}</span>
                                </el-menu-item>
                            </div>
                        </template>
                    </el-submenu>
                </template>
                <template v-else>
                    <el-menu-item :index="item.index" :key="item.index">
                        <i :class="item.icon"></i>
                        <span slot="title">{{ item.title }}</span>
                    </el-menu-item>
                </template>
            </template>
        </el-menu>
    </div>
</template>

<script>
    import bus from '../common/bus';
    export default {
        data() {
            return {
                collapse: false,
                items: [
                    {
                        icon: 'el-icon-lx-home',
                        index: 'dashboard',
                        title: '系统首页'
                    },
                    {
                        icon: 'el-icon-lx-cascades',
                        index: '-',
                        title: '超级管理员',
                        subs: [
                            {
                                index: 'AccountManagement',
                                title: '角色管理',
                                icon: 'el-icon-lx-cascades',
                            },
                            {
                                index: 'group',
                                title: '菜单管理'
                            },
                            {
                                index: 'usergroup',
                                title: 'api管理'
                            },
                            {
                                index: 'allocation',
                                title: '用户管理'
                            },
                            {
                                index: 'menuList',
                                title: '操作历史'
                            }
                        ]
                    },
                    {
                        icon: 'el-icon-lx-calendar',
                        index: '3',
                        title: '表单相关',
                        subs: [
                            {
                                index: 'form',
                                title: '基本表单'
                            },
                            {
                                index: 'editor',
                                title: '富文本编辑器'
                            },
                            {
                                index: 'markdown',
                                title: 'markdown编辑器'
                            },
                            // {
                            //     index: '3-2',
                            //     title: '三级菜单',
                            //     subs: [
                            //         {
                            //             index: '1',
                            //             title: 'XX'
                            //         },
                            //         {
                            //             index: '2',
                            //             title: 'XX2'
                            //         }
                            //     ]
                            // },
                            {
                                index: 'upload',
                                title: 'upload上传'
                            }
                        ]
                    },
                    {
                        icon: 'el-icon-tickets',
                        index: 'table',
                        title: '列表相关'
                    },
                    {
                        icon: 'el-icon-pie-chart',
                        index: 'charts',
                        title: 'Echart图表'
                    },
                    {
                        icon: 'el-icon-lx-copy',
                        index: 'tabs',
                        title: '消息中心'
                    },
                    {
                        icon: 'el-icon-rank',
                        index: '6',
                        title: '拖拽组件',
                        subs: [
                            {
                                index: 'drag',
                                title: '拖拽列表'
                            },
                            {
                                index: 'dialog',
                                title: '拖拽弹框'
                            }
                        ]
                    },
                    {
                        icon: 'el-icon-lx-emoji',
                        index: 'icon',
                        title: '自定义图标'
                    },
                    {
                        icon: 'el-icon-lx-global',
                        index: 'i18n',
                        title: '国际化功能'
                    },
                    {
                        icon: 'el-icon-lx-warn',
                        index: '7',
                        title: '错误处理',
                        subs: [
                            {
                                index: 'permission',
                                title: '权限测试'
                            },
                            {
                                index: '404',
                                title: '404页面'
                            }
                        ]
                    },
                    {
                        icon: 'el-icon-setting',
                        index: 'build',
                        title: '项目不同环境打包相关'
                    },
                ]
            }
        },
        computed:{
            onRoutes(){
                return this.$route.path.replace('/','');
            }
        },
        created(){
            this.menu();
            // 通过 Event Bus 进行组件间通信，来折叠侧边栏
            bus.$on('collapse', msg => {
                this.collapse = msg;
            })
        },
        methods: {
            menu () {
                const self = this       
                self.$http.get({
                    url: 'user/menu',
                    data: {},
                    success (res) {
                        if(res.code == 200){
                            // self.items = res.data;
                        } else {
                            self.$Notice.error({
                                title: '系统提示！',
                                desc: res.msg
                            })
                        }
                    }
                })           
            },
        }
    }
</script>

<style lang="scss" scoped>
.sidebar{
    display: block;
    position: absolute;
    left: -1px;
    top: 70px;
    bottom:0;
    overflow-y: scroll;
    z-index: 2000;
}
.sidebar::-webkit-scrollbar{
    width: 0;
}
.sidebar-el-menu:not(.el-menu--collapse){
    width: 220px;
}
.sidebar > ul {
    height:100%;
}
.aser .el-menu-item.is-active {
    background-color: #1890ff !important;
}
.entr {
    background: #000408 !important;
}
.el-submenu__title:hover {
    span {
        
        color: white !important;
    }
    i {
        color: white;
    }
}
.el-menu-item:hover{
    color: white !important;
    i {
        color: white;
    }
}
.el-submenu.is-active {
    #icontent {
        // color: white;
        color: white !important;
    }
}
</style>
