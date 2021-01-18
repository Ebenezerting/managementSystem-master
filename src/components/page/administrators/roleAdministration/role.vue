<template>
<!-- 角色管理 -->
    <div class="role">
        <div style="position: absolute;top:-52px;left:60px;" class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>超级管理员</el-breadcrumb-item>
                <el-breadcrumb-item>角色管理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="role-header">
                <el-button @click="addAuthority('0')" type="primary">新增角色</el-button>
            </div>
            <el-table
                v-loading="loadingTable"
                element-loading-text="拼命加载中"
                :data="tableData"
                :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
                border
                row-key="authorityId"
                stripe
                style="width: 100%"
                >
                <el-table-column label="角色id" prop="authorityId" align="left"></el-table-column>
                <el-table-column label="角色名称" prop="authorityName" align="center"></el-table-column>
                <el-table-column fixed="right" label="操作" align="center">
                    <template slot-scope="scope">
                    <el-button @click="opdendrawer(scope.row)" size="mini" type="primary">设置权限</el-button>
                        <!-- <el-button
                            @click="addAuthority(scope.row.authorityId)"
                            icon="el-icon-plus"
                            size="mini"
                            type="primary"
                        >新增子角色</el-button> -->
                        <el-button
                            @click="copyAuthority(scope.row)"
                            icon="el-icon-copy-document"
                            size="mini"
                            type="primary"
                        >拷贝</el-button>
                        <el-button
                            @click="editAuthority(scope.row)"
                            icon="el-icon-edit"
                            size="mini"
                            type="primary"
                        >编辑</el-button>
                        <el-button
                            @click="deleteAuth(scope.row)"
                            icon="el-icon-delete"
                            size="mini"
                            type="danger"
                        >删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <!-- 新增角色弹窗 -->
        <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
            <el-form :model="form" :rules="rules" ref="authorityForm">
                <el-form-item label="父级角色" prop="parentId">
                <el-cascader
                    :disabled="dialogType=='add'"
                    :options="AuthorityOption"
                    :props="{ checkStrictly: true,label:'authorityName',value:'authorityId',disabled:'disabled',emitPath:false}"
                    :show-all-levels="false"
                    filterable
                    v-model="form.parentId"
                ></el-cascader>
                </el-form-item>
                <el-form-item label="角色ID" prop="authorityId">
                    <el-input :disabled="dialogType=='edit'" autocomplete="off" v-model="form.authorityId"></el-input>
                </el-form-item>
                <el-form-item label="角色名称" prop="authorityName">
                    <el-input autocomplete="off" v-model="form.authorityName"></el-input>
                </el-form-item>
            </el-form>
            <div class="dialog-footer" slot="footer">
                <el-button @click="closeDialog">取 消</el-button>
                <el-button @click="enterDialog" type="primary">确 定</el-button>
            </div>
        </el-dialog>
        <!-- 权限 -->
        <el-drawer :visible.sync="drawer" :with-header="false" size="40%" title="角色配置" v-if="drawer">
            <el-tabs :before-leave="autoEnter" class="role-box" type="border-card">
                <el-tab-pane label="角色菜单">
                    <Menus :row="activeRow" ref="menus" />
                </el-tab-pane>
                <el-tab-pane label="角色api">
                    <apis :row="activeRow" ref="apis" />
                </el-tab-pane>
                <!-- <el-tab-pane label="资源权限">
                    <Datas :authority="tableData" :row="activeRow" ref="datas" />
                </el-tab-pane> -->
            </el-tabs>
        </el-drawer>
    </div>
</template>

<script>
import { mapGetters } from 'vuex'
import Menus from "../roleAdministration/components/menus";
import Apis from "../roleAdministration/components/apis";
import Datas from "../roleAdministration/components/datas";
import infoList from '@/mixins/infoList'

export default {
    name: 'role',
    mixins: [infoList],
    components: {
        Menus,
        Apis,
        Datas
    },
    data(){
        var mustUint = (rule, value, callback) => {
            if (!(/^[0-9]*[1-9][0-9]*$/).test(value)){
            return  callback(new Error("请输入正整数"));
            } 
            return  callback()
        };
        return {
            listApi: 'authority/getAuthorityList',
            AuthorityOption: [
                {
                    authorityId: "0",
                    authorityName: "根角色"
                }
            ],
            dialogTitle: "新增角色",
            dialogFormVisible: false,
            copyForm: {},
            dialogType: "add",
            form: {
                authorityId: "",
                authorityName: "",
                parentId: "0"
            },
            rules: {
                authorityId: [
                    { required: true, message: "请输入角色ID", trigger: "blur" },
                    {validator: mustUint, trigger: 'blur'  }
                ],
                authorityName: [
                    { required: true, message: "请输入角色名", trigger: "blur" }
                ],
                parentId: [
                    { required: true, message: "请选择请求方式", trigger: "blur" }
                ]
            },
            drawer: false,
            activeRow: {},
        }
    },
    computed:{
        ...mapGetters('user', ['jurisdictionInfo']),
        newlist(){
            return this.jurisdictionInfo
        }
    },
    created() {
        this.pageSize = 999;
        this.getTableData();
    },
    methods:{
        autoEnter(activeName, oldActiveName) {
            const paneArr = ["menus", "apis", "datas"];
            if (oldActiveName) {
                if (this.$refs[paneArr[oldActiveName]].needConfirm) {
                    this.$refs[paneArr[oldActiveName]].enterAndNext();
                    this.$refs[paneArr[oldActiveName]].needConfirm = false;
                }
            }
        },
        closeDialog() {
            this.initForm();
            this.dialogFormVisible = false;
            this.apiDialogFlag = false;
        },
        enterDialog() {
            let vm = this;
            if (vm.form.authorityId == "0") {
                vm.$message({
                    type: "error",
                    message: "角色id不能为0"
                });
                return false;
            }
            vm.$refs.authorityForm.validate(async valid => {
                if (valid) {
                    switch (vm.dialogType) {
                        case "add":
                        {
                            vm.$http.postLogin({
                                url: 'authority/createAuthority',
                                data: vm.form,
                                success(res) {
                                    if (res.code == 0) {
                                        vm.$message({
                                            type: "success",
                                            message: "添加成功!"
                                        });
                                        vm.getTableData();
                                        vm.closeDialog();
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
                        break;
                        case "edit":
                        {
                            vm.$http.put({
                                url: 'authority/updateAuthority',
                                data: vm.form,
                                success(res) {
                                    if (res.code == 0) {
                                        vm.$message({
                                            type: "success",
                                            message: "编辑成功!"
                                        });
                                        vm.getTableData();
                                        vm.closeDialog();
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
                        break;
                        case "copy": {
                            const data = {
                                authority: {
                                    authorityId: "string",
                                    authorityName: "string",
                                    datauthorityId: [],
                                    parentId: "string"
                                },
                                oldAuthorityId: 0
                            };
                            data.authority.authorityId = vm.form.authorityId;
                            data.authority.authorityName = vm.form.authorityName;
                            data.authority.parentId = vm.form.parentId;
                            data.authority.dataAuthorityId = vm.copyForm.dataAuthorityId;
                            data.oldAuthorityId = vm.copyForm.authorityId;
                            vm.$http.postLogin({
                                url: 'authority/copyAuthority',
                                data: data,
                                success(res) {
                                    if (res.code == 0) {
                                        vm.$message({
                                            type: "success",
                                            message: "拷贝成功！!"
                                        });
                                        vm.getTableData();
                                        vm.closeDialog();
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
                    // vm.initForm();
                    // vm.dialogFormVisible = false;
                }
            });
        },
        // 初始化表单
        initForm() {
            if (this.$refs.authorityForm) {
                this.$refs.authorityForm.resetFields();
            }
            this.form = {
                authorityId: "",
                authorityName: "",
                parentId: "0"
            };
        },
        // 拷贝角色
        copyAuthority(row) {
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/authority/copyAuthority'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            if(hide != '') {
                this.setOptions();
                this.dialogTitle = "拷贝角色";
                this.dialogType = "copy";
                for (let k in this.form) {
                    this.form[k] = row[k];
                }
                this.copyForm = row;
                this.dialogFormVisible = true;
            } else {
                this.$notify({
                    title: '警告',
                    message: '拷贝角色权限不足！',
                    type: 'warning'
                })
            }
        },
        // 关闭窗口
        closeDialog() {
            this.initForm();
            this.dialogFormVisible = false;
            this.apiDialogFlag = false;
        },
        // 增加角色
        addAuthority(parentId) {
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
                this.initForm();
                this.dialogTitle = "新增角色";
                this.dialogType = "add";
                this.form.parentId = parentId;
                this.setOptions();
                this.dialogFormVisible = true;
            } else {
                this.$notify({
                    title: '警告',
                    message: '新增角色权限不足！',
                    type: 'warning'
                })
            }
        },
        // 编辑角色
        editAuthority(row) {
            let hide = [];
            let array = this.jurisdictionInfo;
            console.log('array',array)
            let url = '/authority/updateAuthority'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            if(hide != '') {
                this.setOptions();
                this.dialogTitle = "编辑角色";
                this.dialogType = "edit";
                for (let key in this.form) {
                    this.form[key] = row[key];
                }
                this.setOptions();
                this.dialogFormVisible = true;
            } else {
                this.$notify({
                    title: '警告',
                    message: '编辑角色权限不足！',
                    type: 'warning'
                })
            }
        },
        // 删除角色
        deleteAuth(row) {
            let vm = this;
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/authority/deleteAuthority'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            if(hide != '') {
                vm.$confirm("此操作将永久删除该角色, 是否继续?", "提示", {
                    confirmButtonText: "确定",
                    cancelButtonText: "取消",
                    type: "warning"
                })
                .then(async () => {
                    vm.$http.DELETE({
                        url: 'authority/deleteAuthority',
                        data: {
                            authorityId: row.authorityId
                        },
                        success(res) {
                            if (res.code == 0) {
                                vm.$message({
                                    type: "success",
                                    message: "删除成功!"
                                });
                                vm.getTableData();
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
                })
                .catch(() => {
                    vm.$message({
                        type: "info",
                        message: "已取消删除"
                    });
                });
            } else {
                vm.$notify({
                    title: '警告',
                    message: '删除角色权限不足！',
                    type: 'warning'
                })
            }
        },
        // 级联逻辑
        setOptions() {
            this.AuthorityOption = [
                {
                authorityId: "0",
                authorityName: "根角色"
                }
            ];
            this.setAuthorityOptions(this.tableData, this.AuthorityOption, false);
        },
        setAuthorityOptions(AuthorityData, optionsData, disabled) {
            this.form.authorityId = String(this.form.authorityId);
            AuthorityData &&
            AuthorityData.map(item => {
            if (item.children && item.children.length) {
                const option = {
                authorityId: item.authorityId,
                authorityName: item.authorityName,
                disabled: disabled || item.authorityId == this.form.authorityId,
                children: []
                };
                this.setAuthorityOptions(
                item.children,
                option.children,
                disabled || item.authorityId == this.form.authorityId
                );
                optionsData.push(option);
            } else {
                const option = {
                authorityId: item.authorityId,
                authorityName: item.authorityName,
                disabled: disabled || item.authorityId == this.form.authorityId
                };
                optionsData.push(option);
            }
            });
        },
        // 权限
        opdendrawer(row) {
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/authority/setDataAuthority'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            if(hide != '') {
                this.drawer = true;
                this.activeRow = row;
            } else {
                this.$notify({
                    title: '警告',
                    message: '设置角色权限不足！',
                    type: 'warning'
                })
            }
        },
    }
}
</script>

<style lang="scss">
.role {
    .role-header {
        text-align: right;
        margin-bottom: 10px;
    }
}
.role-box {
    .el-tabs__content {
        height: calc(100vh - 150px);
        overflow: auto;
    }
}
</style>