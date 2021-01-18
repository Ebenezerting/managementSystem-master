<template>
<!-- 首页 -->
    <div>
        <div style="position: absolute;top:-52px;left:60px;" class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>首页</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-card shadow="hover" class="mgb20" style="height:252px;">
                        <div class="user-info">
                            <img src="../../assets/img/img.jpg" class="user-avator" alt />
                            <div class="user-info-cont">
                                <div class="user-info-name">{{name}}</div>
                                <div>{{role}}</div>
                            </div>
                        </div>
                        <div class="user-info-list">
                            上次登录时间：
                            <span>{{ time }}</span>
                        </div>
                        <div class="user-info-list">
                            上次登录地点：
                            <span>东莞</span>
                        </div>
                    </el-card>
                </el-col>
                <el-col :span="16">
                    <el-card shadow="hover" style="height:403px;">
                        <div slot="header" class="clearfix">
                            <span>待办事项</span>
                            <el-button style="float: right; padding: 3px 0" type="text">添加</el-button>
                        </div>
                        <el-table :show-header="false" :data="todoList" style="width:100%;">
                            <el-table-column width="40">
                                <template slot-scope="scope">
                                    <el-checkbox v-model="scope.row.status"></el-checkbox>
                                </template>
                            </el-table-column>
                            <el-table-column>
                                <template slot-scope="scope">
                                    <div
                                        class="todo-item"
                                        :class="{'todo-item-del': scope.row.status}"
                                    >{{scope.row.title}}</div>
                                </template>
                            </el-table-column>
                            <el-table-column width="60">
                                <template>
                                    <i class="el-icon-edit"></i>
                                    <i class="el-icon-delete"></i>
                                </template>
                            </el-table-column>
                        </el-table>
                    </el-card>
                </el-col>
            </el-row>
        </div>
    </div>
</template>

<script>
export default {
    name: 'Dashboard',
    data(){
        return {
            time: this.getNowTime(),
            name: localStorage.getItem('ms_username'),
            todoList: [
                {
                    title: '今天要修复100个bug',
                    status: false
                },
                {
                    title: '今天要修复100个bug',
                    status: false
                },
                {
                    title: '今天要写100行代码加几个bug吧',
                    status: false
                },
                {
                    title: '今天要修复100个bug',
                    status: false
                },
                {
                    title: '今天要修复100个bug',
                    status: true
                },
                {
                    title: '今天要写100行代码加几个bug吧',
                    status: true
                }
            ]
        }
    },
    watch: {

    },
    computed: {
        role() {
            return this.name === 'huangzhenzong' ? '超级管理员' : '普通用户';
        }
    },
    mounted() {
    },
    methods:{
        // 获取时间方法
        getNowTime (val) {
            let date = new Date();
            if (val) {
                date = new Date(date.getTime() + 86400000 * val);
            }
            var seperator1 = "-";
            var year = date.getFullYear();
            var month = date.getMonth() + 1;
            var strDate = date.getDate();
            if (month >= 1 && month <= 9) {
                month = "0" + month;
            }
            if (strDate >= 0 && strDate <= 9) {
                strDate = "0" + strDate;
            }
            var currentdate = year + seperator1 + month + seperator1 + strDate;
            return currentdate;
        },
    }
}
</script>

<style lang="scss" scoped>
.el-row {
    margin-bottom: 20px;
}

.grid-content {
    display: flex;
    align-items: center;
    height: 100px;
}

.grid-cont-right {
    flex: 1;
    text-align: center;
    font-size: 14px;
    color: #999;
}

.grid-num {
    font-size: 30px;
    font-weight: bold;
}

.grid-con-icon {
    font-size: 50px;
    width: 100px;
    height: 100px;
    text-align: center;
    line-height: 100px;
    color: #fff;
}

.grid-con-1 .grid-con-icon {
    background: rgb(45, 140, 240);
}

.grid-con-1 .grid-num {
    color: rgb(45, 140, 240);
}

.grid-con-2 .grid-con-icon {
    background: rgb(100, 213, 114);
}

.grid-con-2 .grid-num {
    color: rgb(45, 140, 240);
}

.grid-con-3 .grid-con-icon {
    background: rgb(242, 94, 67);
}

.grid-con-3 .grid-num {
    color: rgb(242, 94, 67);
}

.user-info {
    display: flex;
    align-items: center;
    padding-bottom: 20px;
    border-bottom: 2px solid #ccc;
    margin-bottom: 20px;
}

.user-avator {
    width: 120px;
    height: 120px;
    border-radius: 50%;
}

.user-info-cont {
    padding-left: 50px;
    flex: 1;
    font-size: 14px;
    color: #999;
}

.user-info-cont div:first-child {
    font-size: 30px;
    color: #222;
}

.user-info-list {
    font-size: 14px;
    color: #999;
    line-height: 25px;
}

.user-info-list span {
    margin-left: 70px;
}

.mgb20 {
    margin-bottom: 20px;
}

.todo-item {
    font-size: 14px;
}

.todo-item-del {
    text-decoration: line-through;
    color: #999;
}
</style>