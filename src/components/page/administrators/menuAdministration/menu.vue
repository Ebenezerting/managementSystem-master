<template>
<!-- 菜单管理 -->
    <div>
        <div style="position: absolute;top:-52px;left:60px;" class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>超级管理员</el-breadcrumb-item>
                <el-breadcrumb-item>菜单管理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="button-box clearflex">
                <el-button @click="addMenu('0')" type="primary">新增根菜单</el-button>
            </div>
            <!-- 由于此处菜单跟左侧列表一一对应所以不需要分页 pageSize默认999 -->
            <el-table v-loading="loadingTable" element-loading-text="拼命加载中" :data="tableData" border row-key="ID">
                <el-table-column label="ID" min-width="100" prop="ID"></el-table-column>
                <el-table-column label="路由Name" min-width="160" prop="name" align="center"></el-table-column>
                <el-table-column label="路由Path" min-width="160" prop="path" align="center"></el-table-column>
                <el-table-column label="是否隐藏" min-width="100" prop="hidden" align="center">
                    <template slot-scope="scope">
                    <span>{{scope.row.hidden?"隐藏":"显示"}}</span>
                    </template>
                </el-table-column>
                <el-table-column label="父节点" min-width="90" prop="parentId" align="center"></el-table-column>
                <el-table-column label="排序" min-width="70" prop="sort" align="center"></el-table-column>
                <el-table-column label="文件路径" min-width="300" prop="component" align="center"></el-table-column>
                <el-table-column label="展示名称" min-width="120" prop="authorityName" align="center">
                    <template slot-scope="scope">
                    <span>{{scope.row.meta.title}}</span>
                    </template>
                </el-table-column>
                <el-table-column label="图标" min-width="140" prop="authorityName" align="center">
                    <template slot-scope="scope">
                    <i :class="`el-icon-${scope.row.meta.icon}`"></i>
                    <span>{{scope.row.meta.icon}}</span>
                    </template>
                </el-table-column>
                <el-table-column fixed="right" label="操作" width="300" align="center">
                    <template slot-scope="scope">
                    <el-button
                        @click="addMenu(scope.row.ID)"
                        size="small"
                        type="primary"
                        icon="el-icon-edit"
                    >添加子菜单</el-button>
                    <el-button
                        @click="editMenu(scope.row.ID)"
                        size="small"
                        type="primary"
                        icon="el-icon-edit"
                    >编辑</el-button>
                    <el-button
                        @click="deleteMenu(scope.row.ID)"
                        size="small"
                        type="danger"
                        icon="el-icon-delete"
                    >删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </div>
        <!-- 弹窗 -->
        <el-dialog :before-close="handleClose" :title="dialogTitle" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
            <el-form
                :inline="true"
                :model="form"
                :rules="rules"
                label-position="top"
                label-width="85px"
                ref="menuForm"
            >
                <el-form-item label="路由name" prop="path" style="width:30%">
                <el-input
                    @change="changeName"
                    autocomplete="off"
                    placeholder="唯一英文字符串"
                    v-model="form.name"
                ></el-input>
                </el-form-item>
                <el-form-item prop="path" style="width:30%">
                <div style="display:inline-block" slot="label">
                    路由path
                    <el-checkbox style="float:right;margin-left:20px;" v-model="checkFlag">添加参数</el-checkbox>
                </div>
                <el-input
                    :disabled="!checkFlag"
                    autocomplete="off"
                    placeholder="建议只在后方拼接参数"
                    v-model="form.path"
                ></el-input>
                </el-form-item>
                <el-form-item label="是否隐藏" style="width:30%">
                <el-select placeholder="是否在列表隐藏" v-model="form.hidden">
                    <el-option :value="false" label="否"></el-option>
                    <el-option :value="true" label="是"></el-option>
                </el-select>
                </el-form-item>
                <el-form-item label="父节点Id" style="width:30%">
                <el-cascader
                    :disabled="!this.isEdit"
                    :options="menuOption"
                    :props="{ checkStrictly: true,label:'title',value:'ID',disabled:'disabled',emitPath:false}"
                    :show-all-levels="false"
                    filterable
                    v-model="form.parentId"
                ></el-cascader>
                </el-form-item>
                <el-form-item label="文件路径" prop="component" style="width:30%">
                <el-input autocomplete="off" v-model="form.component"></el-input>
                </el-form-item>
                <el-form-item label="展示名称" prop="meta.title" style="width:30%">
                <el-input autocomplete="off" v-model="form.meta.title"></el-input>
                </el-form-item>
                <el-form-item label="图标" prop="meta.icon" style="width:30%">
                <icon :meta="form.meta">
                    <template slot="prepend">el-icon-</template>
                </icon>
                </el-form-item>
                <el-form-item label="排序标记" prop="sort" style="width:30%">
                <el-input autocomplete="off" v-model.number="form.sort"></el-input>
                </el-form-item>
                <el-form-item label="keepAlive" prop="meta.keepAlive" style="width:30%">
                <el-select placeholder="是否keepAlive缓存页面" v-model="form.meta.keepAlive">
                    <el-option :value="false" label="否"></el-option>
                    <el-option :value="true" label="是"></el-option>
                </el-select>
                </el-form-item>
            </el-form>
            <div class="warning">新增菜单需要在角色管理内配置权限才可使用</div>
            <div>
                <el-button
                size="small"
                type="primary"
                icon="el-icon-edit"
                @click="addParameter(form)"
                >新增菜单参数</el-button>
                <el-table :data="form.parameters" stripe style="width: 100%">
                <el-table-column prop="type" label="参数类型" width="180">
                    <template slot-scope="scope">
                    <el-select v-model="scope.row.type" placeholder="请选择">
                        <el-option key="query" value="query" label="query"></el-option>
                        <el-option key="params" value="params" label="params"></el-option>
                    </el-select>
                    </template>
                </el-table-column>
                <el-table-column prop="key" label="参数key" width="180">
                    <template slot-scope="scope">
                    <div>
                        <el-input v-model="scope.row.key"></el-input>
                    </div>
                    </template>
                </el-table-column>
                <el-table-column prop="value" label="参数值">
                    <template slot-scope="scope">
                    <div>
                        <el-input v-model="scope.row.value"></el-input>
                    </div>
                    </template>
                </el-table-column>
                <el-table-column>
                    <template slot-scope="scope">
                    <div>
                        <el-button
                        type="danger"
                        size="small"
                        icon="el-icon-delete"
                        @click="deleteParameter(form.parameters,scope.$index)"
                        >删除</el-button>
                    </div>
                    </template>
                </el-table-column>
                </el-table>
            </div>
            <div class="dialog-footer" slot="footer">
                <el-button @click="closeDialog">取 消</el-button>
                <el-button @click="enterDialog" type="primary">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import { mapGetters } from 'vuex'
import infoList from "@/mixins/infoList";
import icon from "../menuAdministration/icon";
export default {
    name: 'menus',
    mixins: [infoList],
    components: {
        icon
    },
    data(){
        return {
            checkFlag: false,
            listApi: 'menu/getMenuList',
            dialogFormVisible: false,
            dialogTitle: "新增菜单",
            menuOption: [
                {
                ID: "0",
                title: "根菜单"
                }
            ],
            form: {
                ID: 0,
                path: "",
                name: "",
                hidden: "",
                parentId: "",
                component: "",
                meta: {
                title: "",
                icon: "",
                defaultMenu: false,
                keepAlive: false
                },
                parameters: []
            },
            rules: {
                path: [{ required: true, message: "请输入菜单name", trigger: "blur" }],
                component: [
                { required: true, message: "请输入文件路径", trigger: "blur" }
                ],
                "meta.title": [
                { required: true, message: "请输入菜单展示名称", trigger: "blur" }
                ]
            },
            isEdit: false,
            test: ""
        }
    },
    computed:{
        ...mapGetters('user', ['jurisdictionInfo']),
        newlist(){
            return this.jurisdictionInfo
        }
    },
    async created() {
        this.pageSize = 999;
        await this.getTableData();
    },
    methods:{
        addParameter(form) {
            if (!form.parameters){
                this.$set(form,"parameters",[])
            }
            form.parameters.push({
                type: "query",
                key: "",
                value: ""
            });
            },
        deleteParameter(parameters, index) {
            parameters.splice(index, 1);
        },
        changeName() {
            this.form.path = this.form.name;
        },
        setOptions() {
            this.menuOption = [
                {
                    ID: "0",
                    title: "根目录"
                }
            ];
            this.setMenuOptions(this.tableData, this.menuOption, false);
        },
        setMenuOptions(menuData, optionsData, disabled) {
            menuData &&
            menuData.map(item => {
                if (item.children && item.children.length) {
                    const option = {
                    title: item.meta.title,
                    ID: String(item.ID),
                    disabled: disabled || item.ID == this.form.ID,
                    children: []
                    };
                    this.setMenuOptions(
                    item.children,
                    option.children,
                    disabled || item.ID == this.form.ID
                    );
                    optionsData.push(option);
                } else {
                    const option = {
                    title: item.meta.title,
                    ID: String(item.ID),
                    disabled: disabled || item.ID == this.form.ID
                    };
                    optionsData.push(option);
                }
            });
        },
        handleClose(done) {
            this.initForm();
            done();
        },
        // 懒加载子菜单
        // load(tree, treeNode, resolve) {
        //     resolve([
        //         {
        //         id: 31,
        //         date: "2016-05-01",
        //         name: "王小虎",
        //         address: "上海市普陀区金沙江路 1519 弄"
        //         },
        //         {
        //         id: 32,
        //         date: "2016-05-01",
        //         name: "王小虎",
        //         address: "上海市普陀区金沙江路 1519 弄"
        //         }
        //     ]);
        // },
        // 删除菜单
        deleteMenu(ID) {
            let vm = this;
            vm.$confirm("此操作将永久删除所有角色下该菜单, 是否继续?", "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
            })
            .then(() => {
                vm.$http.postLogin({
                    url: 'menu/deleteBaseMenu',
                    data: {
                        ID: ID
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
                        console.log(res)
                    }
                })
            })
            .catch(() => {
                vm.$message({
                    type: "info",
                    message: "已取消删除"
                });
            });
        },
        // 初始化弹窗内表格方法
        initForm() {
            this.checkFlag = false;
            this.$refs.menuForm.resetFields();
            this.form = {
                ID: 0,
                path: "",
                name: "",
                hidden: "",
                parentId: "",
                component: "",
                meta: {
                    title: "",
                    icon: "",
                    defaultMenu: false,
                    keepAlive: ""
                }
            };
        },
        // 关闭弹窗
        closeDialog() {
            this.initForm();
            this.dialogFormVisible = false;
        },
        // 添加menu
        enterDialog() {
            let vm = this;
            vm.$refs.menuForm.validate(async valid => {
                console.log('vm.form',vm.form)
                if (valid) {
                    if (vm.isEdit) {
                        // 修改menu列表
                        vm.$http.postLogin({
                            url: 'menu/updateBaseMenu',
                            data: vm.form,
                            success(res) {
                                if (res.code == 0) {
                                    vm.$message({
                                        type: "success",
                                        message: "添加成功!"
                                    });
                                    vm.getTableData();
                                    vm.dialogFormVisible = false;
                                } else {
                                    vm.$notify({
                                        title: '警告',
                                        message: res.msg,
                                        type: 'warning'
                                    });
                                }
                            },
                            error(res) {
                                console.log(res)
                            }
                        })
                    } else {
                        // 新增基础menu
                        vm.$http.postLogin({
                            url: 'menu/addBaseMenu',
                            data: vm.form,
                            success(res) {
                                if (res.code == 0) {
                                    vm.$message({
                                        type: "success",
                                        message: "编辑成功!"
                                    });
                                    vm.getTableData();
                                    vm.dialogFormVisible = false;
                                } else {
                                    vm.$notify({
                                        title: '警告',
                                        message: res.msg,
                                        type: 'warning'
                                    });
                                }
                            },
                            error(res) {
                                console.log(res)
                            }
                        })
                    }
                }
            });
        },
        // 添加菜单方法，id为 0则为添加根菜单
        addMenu(id) {
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/menu/addBaseMenu'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            if(hide != '') {
                this.dialogTitle = "新增菜单";
                this.form.parentId = String(id);
                this.isEdit = false;
                this.setOptions();
                this.dialogFormVisible = true;
            } else {
                this.$notify({
                    title: '警告',
                    message: '新增菜单权限不足！',
                    type: 'warning'
                })
            }
        },
        // 修改菜单方法
        editMenu(id) {
            let vm = this;
            vm.dialogTitle = "编辑菜单";
            vm.$http.postLogin({
                url: 'menu/getBaseMenuById',
                data: {
                    id: id
                },
                success(res) {
                    if (res.code == 0) {
                        vm.form = res.data.menu;
                        vm.isEdit = true;
                        vm.setOptions();
                        vm.dialogFormVisible = true;
                    } else {
                        vm.$notify({
                            title: '警告',
                            message: res.msg,
                            type: 'warning'
                        });
                    }
                },
                error(res) {
                    console.log(res)
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
.warning {
    color: #dc143c;
}
</style>