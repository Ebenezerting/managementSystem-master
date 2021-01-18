<template>
<!-- api管理 -->
    <div>
        <div style="position: absolute;top:-52px;left:60px;" class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>超级管理员</el-breadcrumb-item>
                <el-breadcrumb-item>api管理</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div>
                <el-form :inline="true" :model="searchInfo" class="demo-form-inline">
                    <el-form-item label="路径">
                    <el-input placeholder="路径" v-model="searchInfo.path"></el-input>
                    </el-form-item>
                    <el-form-item label="描述">
                    <el-input placeholder="描述" v-model="searchInfo.description"></el-input>
                    </el-form-item>
                    <el-form-item label="api组">
                    <el-input placeholder="api组" v-model="searchInfo.apiGroup"></el-input>
                    </el-form-item>
                    <el-form-item label="请求">
                    <el-select clearable placeholder="请选择" v-model="searchInfo.method">
                        <el-option
                        :key="item.value"
                        :label="`${item.label}(${item.value})`"
                        :value="item.value"
                        v-for="item in methodOptions"
                        ></el-option>
                    </el-select>
                    </el-form-item>
                    <el-form-item>
                    <el-button @click="onSubmit" type="primary">查询</el-button>
                    </el-form-item>
                    <el-form-item>
                    <el-button @click="openDialog('addApi')" type="primary">新增api</el-button>
                    </el-form-item>
                </el-form>
            </div>
            <el-table v-loading="loadingTable" element-loading-text="拼命加载中" :data="tableData" @sort-change="sortChange" border stripe>
                <el-table-column label="id" min-width="60" prop="ID" sortable="custom" align="center"></el-table-column>
                <el-table-column label="api路径" min-width="150" prop="path" sortable="custom" align="center"></el-table-column>
                <el-table-column label="api分组" min-width="150" prop="apiGroup" sortable="custom" align="center"></el-table-column>
                <el-table-column label="api简介" min-width="150" prop="description" sortable="custom" align="center"></el-table-column>
                <el-table-column label="请求" min-width="150" prop="method" sortable="custom" align="center">
                    <template slot-scope="scope">
                    <div>
                        {{scope.row.method}}
                        <el-tag
                        :key="scope.row.methodFiletr"
                        :type="scope.row.method|tagTypeFiletr"
                        effect="dark"
                        size="mini"
                        >{{scope.row.method|methodFiletr}}</el-tag>
                        <!-- {{scope.row.method|methodFiletr}} -->
                    </div>
                    </template>
                </el-table-column>

                <el-table-column fixed="right" label="操作" width="200" align="center">
                    <template slot-scope="scope">
                    <el-button @click="editApi(scope.row)" size="small" type="primary" icon="el-icon-edit">编辑</el-button>
                    <el-button @click="deleteApi(scope.row)" size="small" type="danger" icon="el-icon-delete">删除</el-button>
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
        </div>
        <!-- 弹窗 -->
        <el-dialog :before-close="closeDialog" :title="dialogTitle" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
            <el-form :inline="true" :model="form" :rules="rules" label-width="80px" ref="apiForm">
                <el-form-item label="路径" prop="path">
                <el-input autocomplete="off" v-model="form.path"></el-input>
                </el-form-item>
                <el-form-item label="请求" prop="method">
                <el-select placeholder="请选择" v-model="form.method">
                    <el-option
                    :key="item.value"
                    :label="`${item.label}(${item.value})`"
                    :value="item.value"
                    v-for="item in methodOptions"
                    ></el-option>
                </el-select>
                </el-form-item>
                <el-form-item label="api分组" prop="apiGroup">
                <el-input autocomplete="off" v-model="form.apiGroup"></el-input>
                </el-form-item>
                <el-form-item label="api简介" prop="description">
                <el-input autocomplete="off" v-model="form.description"></el-input>
                </el-form-item>
            </el-form>
            <div class="warning">新增Api需要在角色管理内配置权限才可使用</div>
            <div class="dialog-footer" slot="footer">
                <el-button @click="closeDialog">取 消</el-button>
                <el-button @click="enterDialog" type="primary">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
import { mapGetters } from 'vuex'
import infoList from '@/mixins/infoList'
import { toSQLLine } from '@/utils/stringFun'
const methodOptions = [
  {
    value: 'POST',
    label: '创建',
    type: 'success'
  },
  {
    value: 'GET',
    label: '查看',
    type: ''
  },
  {
    value: 'PUT',
    label: '更新',
    type: 'warning'
  },
  {
    value: 'DELETE',
    label: '删除',
    type: 'danger'
  }
]
export default {
    name: 'Api',
    mixins: [infoList],
    data(){
        return {
            listApi: 'api/getApiList',
            dialogFormVisible: false,
            dialogTitle: '新增Api',
            form: {
                path: '',
                apiGroup: '',
                method: '',
                description: ''
            },
            methodOptions: methodOptions,
            type: '',
            rules: {
                path: [{ required: true, message: '请输入api路径', trigger: 'blur' }],
                apiGroup: [
                { required: true, message: '请输入组名称', trigger: 'blur' }
                ],
                method: [
                { required: true, message: '请选择请求方式', trigger: 'blur' }
                ],
                description: [
                { required: true, message: '请输入api介绍', trigger: 'blur' }
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
        this.getTableData()
    },
    filters: {
        methodFiletr(value) {
            const target = methodOptions.filter(item => item.value === value)[0]
            // return target && `${target.label}(${target.value})`
            return target && `${target.label}`
        },
        tagTypeFiletr(value) {
            const target = methodOptions.filter(item => item.value === value)[0]
            return target && `${target.type}`
        }
    },
    methods: {
        // 排序
        sortChange({ prop, order }) {
            if (prop) {
                this.searchInfo.orderKey = toSQLLine(prop)
                this.searchInfo.desc = order == 'descending'
            }
            this.getTableData()
        },
        //条件搜索前端看此方法
        onSubmit() {
            this.page = 1
            this.pageSize = 10
            this.getTableData()
        },
        initForm() {
            this.$refs.apiForm.resetFields()
            this.form= {
                path: '',
                apiGroup: '',
                method: '',
                description: ''
            }
        },
        closeDialog() {
            this.initForm()
            this.dialogFormVisible = false
        },
        openDialog(type) {
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/api/createApi'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            switch (type) {
                case 'addApi':
                if(hide != '') {
                    this.dialogTitlethis = '新增Api'
                } else {
                    this.$notify({
                        title: '警告',
                        message: '新增API限不足！',
                        type: 'warning'
                    })
                    return
                }
                break
                case 'edit':
                this.dialogTitlethis = '编辑Api'
                break
                default:
                break
            }
            this.type = type
            this.dialogFormVisible = true
        },
        editApi(row) {
            let vm = this;
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/api/updateApi'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }           
            if(hide != '') {
                vm.$http.postLogin({
                    url: 'api/getApiById',
                    data: {
                        id: row.ID
                    },
                    success(res) {
                        if (res.code == 0) {
                            vm.form = res.data.api
                            vm.openDialog('edit')
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
            } else {
                this.$notify({
                    title: '警告',
                    message: 'api编辑权限不足！',
                    type: 'warning'
                })
            }
        },
        deleteApi(row) {
            let vm = this;
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/api/deleteApi'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            if(hide != '') {
                vm.$confirm('此操作将永久删除所有角色下该菜单, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                })
                .then(() => {
                    vm.$http.postLogin({
                        url: 'api/deleteApi',
                        data: row,
                        success(res) {
                            if (res.code == 0) {
                                vm.$message({
                                    type: 'success',
                                    message: '删除成功!'
                                })
                                vm.getTableData()
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
                        type: 'info',
                        message: '已取消删除'
                    })
                })
            } else {
                this.$notify({
                    title: '警告',
                    message: '删除用户权限不足！',
                    type: 'warning'
                })
            }
        },
        enterDialog() {
            let vm = this;
            vm.$refs.apiForm.validate(async valid => {
                if (valid) {
                switch (vm.type) {
                    case 'addApi':
                    {
                        vm.$http.postLogin({
                            url: 'api/createApi',
                            data: vm.form,
                            success(res) {
                                if (res.code == 0) {
                                    vm.$message({
                                        type: "success",
                                        message: "添加成功!"
                                    });
                                    vm.getTableData()
                                    vm.closeDialog()
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

                    break
                    case 'edit':
                    {
                        vm.$http.postLogin({
                            url: 'api/updateApi',
                            data: vm.form,
                            success(res) {
                                if (res.code == 0) {
                                    vm.$message({
                                        type: 'success',
                                        message: '编辑成功',
                                        showClose: true
                                    })
                                    vm.getTableData()
                                    vm.closeDialog()
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
                    break
                    default:
                    {
                        vm.$message({
                        type: 'error',
                        message: '未知操作',
                        showClose: true
                        })
                    }
                    break
                }
                }
            })
        }
    }
}
</script>

<style lang="scss" scoped>
.warning {
    color: #dc143c;
}
</style>