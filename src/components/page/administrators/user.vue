<template>
<!-- 用户管理 -->
    <div>
        <div style="position: absolute;top:-52px;left:60px;" class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>超级管理员</el-breadcrumb-item>
                <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="button-box clearflex">
                <el-button @click="addUser" type="primary">新增用户</el-button>
            </div>
            <el-table v-loading="loadingTable" element-loading-text="拼命加载中" :data="tableData" border stripe>
                <el-table-column label="头像" min-width="50" align="center">
                    <template slot-scope="scope">
                    <div :style="{'textAlign':'center'}">
                        <CustomPic :picSrc="scope.row.headerImg" />
                    </div>
                    </template>
                </el-table-column>
                <el-table-column label="uuid" min-width="250" prop="uuid" align="center"></el-table-column>
                <el-table-column label="用户名" min-width="150" prop="userName" align="center"></el-table-column>
                <el-table-column label="昵称" min-width="150" prop="nickName" align="center"></el-table-column>
                <el-table-column label="用户角色" min-width="150" align="center">
                    <template slot-scope="scope">
                    <el-cascader
                        @change="changeAuthority(scope.row)"
                        v-model="scope.row.authority.authorityId"
                        :options="authOptions"
                        :show-all-levels="false"
                        :props="{ checkStrictly: true,label:'authorityName',value:'authorityId',disabled:'disabled',emitPath:false}"
                        filterable
                    ></el-cascader>
                    </template>
                </el-table-column>
                <el-table-column label="操作" min-width="150" align="center">
                    <template slot-scope="scope">
                    <el-popover placement="top" width="160" v-model="scope.row.visible">
                        <p>确定要删除此用户吗</p>
                        <div style="text-align: right; margin: 0">
                        <el-button size="mini" type="text" @click="scope.row.visible = false">取消</el-button>
                        <el-button type="primary" size="mini" @click="deleteUser(scope.row)">确定</el-button>
                        </div>
                        <el-button type="danger" icon="el-icon-delete" size="small" slot="reference">删除</el-button>
                    </el-popover>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination
            :current-page="page"
            :page-size="pageSize"
            :page-sizes="[10, 30, 50, 100]"
            :style="{float:'right',padding:'20px'}"
            :total="total"
            @current-change="handleCurrentChange"
            @size-change="handleSizeChange"
            layout="total, sizes, prev, pager, next, jumper"
            ></el-pagination>

            <el-dialog :visible.sync="addUserDialog" custom-class="user-dialog" title="新增用户">
            <el-form :rules="rules" ref="userForm" :model="userInfo">
                <el-form-item label="用户名" label-width="80px" prop="username">
                <el-input v-model="userInfo.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" label-width="80px" prop="password">
                <el-input v-model="userInfo.password"></el-input>
                </el-form-item>
                <el-form-item label="昵称" label-width="80px" prop="nickName">
                <el-input v-model="userInfo.nickName"></el-input>
                </el-form-item>
                <el-form-item label="头像" label-width="80px">
                <div style="display:inline-block" @click="openHeaderChange">
                    <img class="header-img-box" v-if="userInfo.headerImg" :src="userInfo.headerImg" />
                    <div v-else class="header-img-box">从媒体库选择</div>
                </div>
                </el-form-item>
                <el-form-item label="用户角色" label-width="80px" prop="authorityId">
                <el-cascader
                    v-model="userInfo.authorityId"
                    :options="authOptions"
                    :show-all-levels="false"
                    :props="{ checkStrictly: true,label:'authorityName',value:'authorityId',disabled:'disabled',emitPath:false}"
                    filterable
                ></el-cascader>
                </el-form-item>
            </el-form>
            <div class="dialog-footer" slot="footer">
                <el-button @click="closeAddUserDialog">取 消</el-button>
                <el-button @click="enterAddUserDialog" type="primary">确 定</el-button>
            </div>
            </el-dialog>
            <ChooseImg ref="chooseImg" :target="userInfo" :targetKey="`headerImg`"/>
        </div>
    </div>
</template>

<script>
const path = process.env.VUE_APP_BASE_API;
import infoList from "@/mixins/infoList";
import { mapGetters } from "vuex";
import CustomPic from "@/components/common/customPic";
import ChooseImg from "@/components/common/chooseImg";
export default {
    name: "User",
    mixins: [infoList],
    components: { CustomPic, ChooseImg },
    data(){
        return {
            listApi: 'user/getUserList',
            path: path,
            authOptions: [],
            addUserDialog: false,
            userInfo: {
                username: "",
                password: "",
                nickName: "",
                headerImg: "",
                authorityId: ""
            },
            rules: {
                username: [
                { required: true, message: "请输入用户名", trigger: "blur" },
                { min: 6, message: "最低6位字符", trigger: "blur" }
                ],
                password: [
                { required: true, message: "请输入用户密码", trigger: "blur" },
                { min: 6, message: "最低6位字符", trigger: "blur" }
                ],
                nickName: [
                { required: true, message: "请输入用户昵称", trigger: "blur" }
                ],
                authorityId: [
                { required: true, message: "请选择用户角色", trigger: "blur" }
                ]
            }
        }
    },
    computed:{
        ...mapGetters('user', ['jurisdictionInfo']),
        newlist(){
            return this.jurisdictionInfo
        }
    },
    created() {
        this.getTableData();
        this.accList();
    },
    methods:{
        accList() {
            let vm = this;
            vm.$http.postLogin({
                url: 'authority/getAuthorityList',
                data: {
                    page: 1,
                    pageSize: 999
                },
                success(res) {
                    vm.setOptions(res.data.list);
                },
                error(res) {
                    console.log(res)
                }
            })
        },
        openHeaderChange(){
            this.$refs.chooseImg.open()
        },
        setOptions(authData) {
            this.authOptions = [];
            this.setAuthorityOptions(authData, this.authOptions);
        },
        setAuthorityOptions(AuthorityData, optionsData) {
            AuthorityData &&
            AuthorityData.map(item => {
            if (item.children && item.children.length) {
                const option = {
                authorityId: item.authorityId,
                authorityName: item.authorityName,
                children: []
                };
                this.setAuthorityOptions(item.children, option.children);
                optionsData.push(option);
            } else {
                const option = {
                authorityId: item.authorityId,
                authorityName: item.authorityName
                };
                optionsData.push(option);
            }
            });
        },
        deleteUser(row) {
            let vm = this;
            vm.$http.DELETE({
                url: 'user/deleteUser',
                data: {
                    id: row.ID
                },
                success(res) {
                    if (res.code == 0) {
                        vm.$message({ type: "success", message: "删除成功" });
                        vm.getTableData();
                        row.visible = false;
                    } else {
                        vm.$notify({
                            title: '警告',
                            message: '用户删除' + res.msg,
                            type: 'warning'
                        });
                    }
                },
                error(res) {
                    console.log('失败',res)
                }
            })
        },
        enterAddUserDialog() {
            let vm = this;
            vm.$refs.userForm.validate(valid => {
                if (valid) {
                    vm.$http.postLogin({
                        url: 'user/register',
                        data: vm.userInfo,
                        success(res) {
                            if (res.code == 0) {
                                vm.$message({ type: "success", message: "创建成功" });
                                vm.getTableData();
                                vm.closeAddUserDialog();
                            } else {
                                vm.$notify({
                                    title: '警告',
                                    message: res.msg,
                                    type: 'warning'
                                });
                            }
                        },
                        error(res) {
                            console.log('失败',res)
                        }
                    })
                }
            });
        },
        closeAddUserDialog() {
            this.$refs.userForm.resetFields();
            this.addUserDialog = false;
        },
        handleAvatarSuccess(res) {
            this.userInfo.headerImg = res.data.file.url;
        },
        addUser() {
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/authority/createAuthority'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            if(hide != '') {
                this.addUserDialog = true;
            } else {
                this.$notify({
                    title: '警告',
                    message: '新增用户权限不足！',
                    type: 'warning'
                })
            }
        },
        changeAuthority(row) {
            let vm = this;
            vm.$http.postLogin({
                url: 'user/setUserAuthority',
                data: {
                    uuid: row.uuid,
                    authorityId: row.authority.authorityId
                },
                success(res) {
                    if (res.code == 0) {
                        vm.$message({ type: "success", message: "角色设置成功" });
                    } else {
                        vm.$notify({
                            title: '警告',
                            message: res.msg,
                            type: 'warning'
                        });
                    }
                },
                error(res) {
                    console.log('失败',res)
                }
            })
        }
    }
}
</script>

<style lang="scss" scoped>
.button-box {
    padding-bottom: 10px;
    .el-button {  
        float: right;
    }
}
</style>