<template>
	<div>
		<div class="clearflex">
			<el-button @click="authApiEnter" class="fl-right" size="small" type="primary">确 定</el-button>
		</div>
		<el-tree
			:data="apiTreeData"
			:default-checked-keys="apiTreeIds"
			:props="apiDefaultProps"
			@check="nodeChange"
			default-expand-all
			highlight-current
			node-key="onlyId"
			ref="apiTree"
			show-checkbox
		></el-tree>
	</div>
</template>
<script>

export default {
	name: 'Apis',
	props: {
		row: {
			default: function() {
				return {}
			},
			type: Object
		}
	},
	data() {
		return {
			apiTreeData: [],
			apiTreeIds: [],
			needConfirm:false,
			apiDefaultProps: {
				children: 'children',
				label: 'description'
			}
		}
	},
	created() {
		this.getAllApis();
		this.getPolicyPathByAuthorityId();
	},
	methods: {
		// 获取所有的Api 不分页
		getAllApis() {
            let vm = this;
            vm.$http.postLogin({
                url: 'api/getAllApis',
                data: {},
                success(res) {
					const apis = res.data.apis
					vm.apiTreeData = vm.buildApiTree(apis)
                },
                error(res) {
                    console.log(res)
                }
            })
		},
		// 获取权限列表
		getPolicyPathByAuthorityId() {
			let vm = this;
            vm.$http.postLogin({
                url: 'casbin/getPolicyPathByAuthorityId',
                data: {
					authorityId: vm.row.authorityId
				},
                success(res) {
					vm.activeUserId = vm.row.authorityId
					vm.apiTreeIds = []
					res.data.paths&&res.data.paths.map(item=>{
						vm.apiTreeIds.push("p:"+item.path+"m:"+item.method)
					})
                },
                error(res) {
                    console.log(res)
                }
            })
		},
		nodeChange(){
			this.needConfirm = true
		},
		// 暴露给外层使用的切换拦截统一方法
		enterAndNext(){
			this.authApiEnter()
		},
		// 创建api树方法
		buildApiTree(apis) {
			const apiObj = new Object()
			apis &&
				apis.map(item => {
					item.onlyId = "p:"+item.path+"m:"+item.method
					if (Object.prototype.hasOwnProperty.call(apiObj,item.apiGroup)) {
						apiObj[item.apiGroup].push(item)
					} else {
						Object.assign(apiObj, { [item.apiGroup]: [item] })
					}
				})
			const apiTree = []
			for (const key in apiObj) {
				const treeNode = {
					ID: key,
					description: key + '组',
					children: apiObj[key]
				}
				apiTree.push(treeNode)
			}
			return apiTree
		},
		// 关联关系确定
	 	authApiEnter() {
			let vm = this;
			const checkArr = vm.$refs.apiTree.getCheckedNodes(true)
			var casbinInfos = []
			checkArr&&checkArr.map(item=>{
				var casbinInfo = {
					path:item.path,
					method:item.method
				}
				casbinInfos.push(casbinInfo)
			})
			// 更改角色api权限
            vm.$http.postLogin({
                url: 'casbin/updateCasbin',
                data: {
					authorityId: vm.activeUserId,
					casbinInfos
                },
                success(res) {		
					if (res.code == 0) {
						vm.$message({ type: 'success', message: "api设置成功" })
					}
                },
                error(res) {
                    console.log(res)
                }
            })
		}
	},
}
</script>
<style lang="scss">
</style>