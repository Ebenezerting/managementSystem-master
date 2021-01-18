<template>
	<div>
		<div class="clearflex">
			<el-button @click="relation" class="fl-right" size="small" type="primary">确 定</el-button>
		</div>
		<el-tree
		:data="menuTreeData"
		:default-checked-keys="menuTreeIds"
		:props="menuDefaultProps"
		@check="nodeChange"
		default-expand-all
		highlight-current
		node-key="ID"
		ref="menuTree"
		show-checkbox
		></el-tree>
	</div>
</template>
<script>

export default {
	name: 'Menus',
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
			menuTreeData: [],
			menuTreeIds: [],
			needConfirm:false,
			menuDefaultProps: {
				children: 'children',
				label: function(data){
					return data.meta.title
				}
			}
		}
	},
	created() {
		this.getBaseMenuTree();
		this.getMenuAuthority();
	},
	methods: {
		// 获取所有菜单树
		getBaseMenuTree() {
			let vm = this;
			// 获取所有菜单树
			vm.$http.postLogin({
				url: 'menu/getBaseMenuTree',
				data: {},
				success(res) {
					vm.menuTreeData = res.data.menus
				},
				error(res) {
				}
			})
		},
		// 获取所有菜单树
		getMenuAuthority() {
			let vm = this;
			vm.$http.postLogin({
				url: 'menu/getMenuAuthority',
				data: {
					authorityId: vm.row.authorityId
				},
				success(res) {
					const menus = res.data.menus
					const arr = []
					menus.map(item => {
					  // 防止直接选中父级造成全选
					  if (!menus.some(same => same.parentId === item.menuId)) {
					    arr.push(Number(item.menuId))
					  }
					})
					vm.menuTreeIds = arr
				},
				error(res) {
				}
			})
		},
		nodeChange(){
			this.needConfirm = true
		},
		// 暴露给外层使用的切换拦截统一方法
		enterAndNext(){
			this.relation()
		},
		// 关联树 确认方法
		relation() {
			const checkArr = this.$refs.menuTree.getCheckedNodes(false, true)
			// 添加用户menu关联关系
			let vm = this;
			// 获取所有菜单树
			vm.$http.postLogin({
				url: 'menu/addMenuAuthority',
				data: {
					menus: checkArr,
					authorityId: vm.row.authorityId
				},
				success(res) {
					if (res.code == 0) {
						vm.$message({
							type: 'success',
							message: '菜单设置成功!'
						})
					} else {
						vm.$notify({
							title: '警告',
							message: res.msg,
							type: 'warning'
						});
					}
				},
				error(res) {
				}
			})
		}
	}
}
</script>
<style lang="scss">
</style>