<template>
<!-- 操作历史 -->
    <div>
        <div style="position: absolute;top:-52px;left:60px;" class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>超级管理员</el-breadcrumb-item>
                <el-breadcrumb-item>操作历史</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <el-form :inline="true" :model="searchInfo" class="demo-form-inline">
                <el-form-item label="请求方法">
                <el-input placeholder="搜索条件" v-model="searchInfo.method"></el-input>
                </el-form-item>
                <el-form-item label="请求路径">
                <el-input placeholder="搜索条件" v-model="searchInfo.path"></el-input>
                </el-form-item>
                <el-form-item label="结果状态码">
                <el-input placeholder="搜索条件" v-model="searchInfo.status"></el-input>
                </el-form-item>
                <el-form-item>
                <el-button @click="onSubmit" type="primary">查询</el-button> 
                </el-form-item>
                <el-form-item>
                <el-popover @show="deleModal" placement="top" v-model="deleteVisible" width="160">
                    <p>确定要删除吗？</p>
                    <div style="text-align: right; margin: 0">
                    <el-button @click="deleteVisible = false" size="mini" type="text">取消</el-button>
                    <el-button @click="onDelete" size="mini" type="primary">确定</el-button>
                    </div>
                    <el-button icon="el-icon-delete" size="mini" slot="reference" type="danger">批量删除</el-button>
                </el-popover>
                </el-form-item>
            </el-form>   
            <template>

                <u-table
                    ref="plTable"
                    v-loading="loadingTable"
                    :data="tableData"
                    :height="height"
                    use-virtual
                    showBodyOverflow="title"
                    showHeaderOverflow="title"
                    :row-height="rowHeight"
                    :pagination-show="true"
                    :total="pageForm.total"
                    :page-size="pageForm.pageSize"
                    :page-sizes="[10, 30, 50, 100]"
                    :current-page="pageForm.currentPage"
                    @handlePageSize="handlePageSize">
                    <u-table-column type="selection" width="55" align="center" />
                    <u-table-column label="操作人" width="140" align="center">
                        <template slot-scope="scope">
                            <div>{{scope.row.user.userName}}({{scope.row.user.nickName}})</div>
                        </template>
                    </u-table-column>
                    <u-table-column label="日期" width="180" align="center">
                        <template slot-scope="scope">{{scope.row.CreatedAt|formatDate}}</template>
                    </u-table-column>
                    <u-table-column label="状态码" prop="status" width="120" align="center">
                        <template slot-scope="scope">
                            <div>
                                <el-tag type="success">{{ scope.row.status }}</el-tag>
                            </div>
                            </template>
                    </u-table-column>
                    <u-table-column label="请求ip" prop="ip" width="120" align="center"></u-table-column>
                    <u-table-column label="请求方法" prop="method" width="120" align="center"></u-table-column>
                    <u-table-column label="请求路径" prop="path" width="240" align="center"></u-table-column>
                    <u-table-column label="请求" prop="path" width="80" align="center">
                        <template slot-scope="scope">
                            <div>
                                <el-popover placement="right" trigger="hover" v-if="scope.row.body">
                                <div class="popover-box">
                                    <pre>{{fmtBody(scope.row.body)}}</pre>
                                </div>
                                <i class="el-icon-view" slot="reference"></i>
                                </el-popover>
                                <span v-else>无</span>
                            </div>
                        </template>
                    </u-table-column>
                    <u-table-column label="响应" prop="path" width="80" align="center">
                         <template slot-scope="scope">
                            <div>
                                <el-popover placement="right" trigger="hover" v-if="scope.row.resp">
                                <div class="popover-box">
                                    <pre>{{fmtBody(scope.row.resp)}}</pre>
                                </div>
                                <i class="el-icon-view" slot="reference"></i>
                                </el-popover>
                                <span v-else>无</span>
                            </div>
                        </template>
                    </u-table-column>
                    <u-table-column label="按钮组" align="center">
                        <template slot-scope="scope">
                            <el-popover @show="deleRecord(scope.row)" placement="top" v-model="scope.row.visible" width="160">
                                <p>确定要删除吗？</p>
                                <div style="text-align: right; margin: 0">
                                <el-button @click="scope.row.visible = false" size="mini" type="text">取消</el-button>
                                <el-button @click="deleteSysOperationRecord(scope.row)" size="mini" type="primary">确定</el-button>
                                </div>
                                <el-button icon="el-icon-delete" size="mini" slot="reference" type="danger">删除</el-button>
                            </el-popover>
                        </template>
                    </u-table-column>
                </u-table>
            </template>

            <!-- <el-table
                v-loading="loadingTable"
                element-loading-text="拼命加载中"
                :data="tableData"
                @selection-change="handleSelectionChange"
                border
                ref="multipleTable"
                stripe
                style="width: 100%"
                tooltip-effect="dark">
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column label="操作人" width="140" align="center">
                    <template slot-scope="scope">
                    <div>{{scope.row.user.userName}}({{scope.row.user.nickName}})</div>
                    </template>
                </el-table-column>
                <el-table-column label="日期" width="180" align="center">
                    <template slot-scope="scope">{{scope.row.CreatedAt|formatDate}}</template>
                </el-table-column>
                <el-table-column label="状态码" prop="status" width="120" align="center">
                    <template slot-scope="scope">
                    <div>
                        <el-tag type="success">{{ scope.row.status }}</el-tag>
                    </div>
                    </template>
                </el-table-column>
                <el-table-column label="请求ip" prop="ip" width="120" align="center"></el-table-column>
                <el-table-column label="请求方法" prop="method" width="120" align="center"></el-table-column>
                <el-table-column label="请求路径" prop="path" width="240" align="center"></el-table-column>
                <el-table-column label="请求" prop="path" width="80" align="center">
                    <template slot-scope="scope">
                        <div>
                            <el-popover placement="right" trigger="hover" v-if="scope.row.body">
                            <div class="popover-box">
                                <pre>{{fmtBody(scope.row.body)}}</pre>
                            </div>
                            <i class="el-icon-view" slot="reference"></i>
                            </el-popover>

                            <span v-else>无</span>
                        </div>
                    </template>
                </el-table-column>
                <el-table-column label="响应" prop="path" width="80" align="center">
                <template slot-scope="scope">
                    <div>
                        <el-popover placement="right" trigger="hover" v-if="scope.row.resp">
                        <div class="popover-box">
                            <pre>{{fmtBody(scope.row.resp)}}</pre>
                        </div>
                        <i class="el-icon-view" slot="reference"></i>
                        </el-popover>
                        <span v-else>无</span>
                    </div>
                </template>
            </el-table-column>
            <el-table-column label="按钮组" align="center">
                <template slot-scope="scope">
                    <el-popover @show="deleRecord(scope.row)" placement="top" v-model="scope.row.visible" width="160">
                        <p>确定要删除吗？</p>
                        <div style="text-align: right; margin: 0">
                        <el-button @click="scope.row.visible = false" size="mini" type="text">取消</el-button>
                        <el-button @click="deleteSysOperationRecord(scope.row)" size="mini" type="primary">确定</el-button>
                        </div>
                        <el-button icon="el-icon-delete" size="mini" slot="reference" type="danger">删除</el-button>
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
            ></el-pagination>      -->
        </div>
    </div>
</template>

<script>
import { formatTimeToStr } from '@/utils/date'
// import infoList from '@/mixins/infoList'
import { mapGetters } from "vuex";
export default {
    name: 'operation',
    // mixins: [infoList],
    data(){
        return {
            height: 0,
            rowHeight: 55, // 如果你这里给行高为50，那么你表格行会出现错乱，不要问为啥，因为你可以看看控制台看节点的高是多少，是55，而你这里给50就有问题！
            tableData: [],  
            listApi: 'sysOperationRecord/getSysOperationRecordList',
            pageForm: {
              total: 1000,
              pageSize: 10,
              currentPage: 1
            },
            searchInfo: {},
            dialogFormVisible: false,
            loadingTable: false,
            visible: false,
            deleteVisible: false,
            formData: {
                ip: null,
                method: null,
                path: null,
                status: null,
                latency: null,
                agent: null,
                error_message: null,
                user_id: null
            }
        }
    },
    filters: {
        formatDate: function(time) {
            if (time != null && time != '') {
                var date = new Date(time)
                return formatTimeToStr(date, 'yyyy-MM-dd hh:mm:ss')
            } else {
                return ''
            }
        },
        formatBoolean: function(bool) {
            if (bool != null) {
                return bool ? '是' : '否'
            } else {
                return ''
            }
        }
    },
    mounted () {
        // 不使用分页  下面的这个计算是（假装给个示例）。请结合自己情况去计算，
        // 比如我一个页面有头部，表单高级搜索等盒子，那么我这里就这样计算 : 表格高度 =  浏览器的窗口高度 - 我的头部盒子高度 - 表单高级搜索高度
        // this.height = 浏览器的窗口高度  - (this.$refs.goodsIndex.offsetHeight - this.$refs.OrderSearch.offsetHeight)
        this.height = window.innerHeight - 397
        // 使用分页你得这样去计算高度。加个延迟
        setTimeout(() => {
            // 比如我一个页面有头部，表单高级搜索等盒子，那么我这里就这样计算 : 表格高度 =  浏览器的窗口高度 - 我的头部盒子高度 - 表单高级搜索高度
           // this.height = 浏览器的窗口高度  - (this.$refs.goodsIndex.offsetHeight - this.$refs.OrderSearch.offsetHeight)
           this.height = window.innerHeight - 397
        })
    },
    watch: {
        $route(val){
            if(val.path == '/operation') {
                this.acclist();
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
        this.acclist();
    },
    methods: {
        acclist() {
            let vm = this;
            vm.loadingTable = true
            vm.$http.get({
                url: this.listApi,
                data: {
                    page: vm.pageForm.currentPage,
                    pageSize: vm.pageForm.pageSize,
                    ...this.searchInfo
                },
                success(res) {
                    const table = res
                    if (table.code == 0) {
                        vm.tableData = table.data.list
                        vm.pageForm.total = table.data.total
                        vm.page = table.data.page
                        vm.pageSize = table.data.pageSize
                        vm.loadingTable = false;
                    } else {
                        vm.loadingTable = false;
                        Notification({
                            title: '警告',
                            message: '列表' + table.msg,
                            type: 'warning'
                        });
                    }
                },
                error(res) {
                    vm.loadingTable = false;
                    Notification({
                        title: '警告',
                        message: '后端接口报错！',
                        type: 'warning'
                    });
                    console.log(res)
                }
            })
        },
        //条件搜索前端看此方法
        onSubmit() {
            this.page = 1
            this.pageSize = 10
            this.acclist()
        },
        handleSelectionChange(val) {
            this.multipleSelection = val
        },
        deleModal() {
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/sysOperationRecord/deleteSysOperationRecordByIds'
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            if(hide != '') {

            } else {
                this.deleteVisible = false;
                this.$notify({
                    title: '警告',
                    message: '批量删除权限不足！',
                    type: 'warning'
                })
                return
            }
        },
        onDelete() {
            let vm = this;
            const ids = []
            vm.$refs.plTable.getCheckboxRecords() &&
                vm.$refs.plTable.getCheckboxRecords().map(item => {
                ids.push(item.ID)
            })
            if(ids == '') {
                this.$notify({
                    title: '警告',
                    message: '请先选择删除数据！',
                    type: 'warning'
                })
                return
            }
            vm.$http.DELETE({
                url: 'sysOperationRecord/deleteSysOperationRecordByIds',
                data: {ids},
                success(res) {
                    if (res.code == 0) {
                        vm.$message({
                            type: 'success',
                            message: '删除成功'
                        })
                        vm.deleteVisible = false
                        vm.acclist()
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
        },
        deleRecord(data) {
            let hide = [];
            let array = this.jurisdictionInfo;
            let url = '/sysOperationRecord/deleteSysOperationRecord'
            console.log(array)
            if(array != '') {
                array.paths.forEach(item => {       // 调用了forEach 进行遍历
                    if(item.path.indexOf(url) !=- 1){ // 进行条件判断，看是否包含
                        hide.push(item.path)        //由于 forEach 没有返回数组 所以要进行push 到新的数组并返回、
                    }
                });
            }
            console.log(hide)
            if(hide != '') {

            } else {
                data.visible = false;
                this.$notify({
                    title: '警告',
                    message: '删除权限不足！',
                    type: 'warning'
                })
                return
            }
        },
        deleteSysOperationRecord(row) {
            let vm = this;
            vm.visible = false
            vm.$http.DELETE({
                url: 'sysOperationRecord/deleteSysOperationRecord',
                data: {
                    ID: row.ID
                },
                success(res) {
                    if (res.code == 0) {
                        vm.$message({
                            type: 'success',
                            message: '删除成功'
                        })  
                        vm.acclist()
                    } else {
                        vm.$notify({
                            title: '警告',
                            message: '操作历史删除' + res.msg,
                            type: 'warning'
                        });
                    }
                },
                error(res) {
                    console.log('失败',res)
                }
            })
        },
        fmtBody(value){
            try{
                return JSON.parse(value)
            }catch (err){
                return  value
            }
        },
        // 分页事件
        handlePageSize ({page, size}) {
            this.pageForm.currentPage = page;
            this.pageForm.pageSize = size;
            this.acclist()
        }
    }
}
</script>

<style lang="scss" scoped>
.table-expand {
    padding-left: 60px;
    font-size: 0;
    label {
        width: 90px;
        color: #99a9bf;
        .el-form-item {
            margin-right: 0;
            margin-bottom: 0;
            width: 50%;
        }
    }
}
.popover-box {
    background: #112435;
    color: #f08047;
    width: 420px;
    height: 600px;
    overflow: auto;
}
.popover-box::-webkit-scrollbar {
    display: none; /* Chrome Safari */
}
</style>