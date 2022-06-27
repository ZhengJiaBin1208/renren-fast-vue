<template>
    <div>
        <el-tree :data="data" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId"
            :default-expanded-keys=expandKeys
            draggable="true"
            >
            <span class="custom-tree-node" slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span>
                    <el-button v-if="data.catLevel <= 2" type="text" size="mini" @click="() => append(data)">
                        添加
                    </el-button>
                    <el-button type="text" size="mini" @click="() => edit(data)">
                        更新
                    </el-button>
                    <el-button v-if="data.childrens.length == 0" type="text" size="mini"
                        @click="() => remove(node, data)">
                        删除
                    </el-button>
                </span>
            </span>
        </el-tree>
        <!-- 添加弹出框 -->
        <el-dialog :title="dialogType ? '新增' : '更新'" :visible.sync="dialogVisible" width="30%"
            :close-on-click-modal="false">
            <el-form :model="categoryForm">
                <el-form-item label="类别名称" :label-width="formLabelWidth">
                    <el-input v-model="categoryForm.name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="类别图标" :label-width="formLabelWidth">
                    <el-input v-model="categoryForm.icon" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="计量单位" :label-width="formLabelWidth">
                    <el-input v-model="categoryForm.productUnit" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="submitForm">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
/* eslint-disable */
export default {
    data() {
        return {
            // 增加或更新标志位，true新增，false更新
            dialogType: false,
            formLabelWidth: '120px',
            data: [],
            expandKeys: [],
            dialogVisible: false,
            categoryForm: {name:null,icon:null,productUnit:null,showStatus:1,sort:0,catId:null,catLevel:1},
            defaultProps: {
                children: 'childrens',
                label: 'name'
            }
        };
    },
    methods: {
        getCategory() {
            this.$http({
                url: this.$http.adornUrl('/product/category/listTree'),
                method: 'get'
            }).then(({ data }) => {
                console.log("成功获取的类别数据: ", data.data)
                this.data = data.data
            });
        }, append(data) {
            // 添加时根据父级目录初始化
            // 打开弹出窗口
            this.dialogVisible = true;
            // console.log("添加", data);
            // 添加类别对应的父菜单
            this.categoryForm.parentCid = data.catId;
            // 设置添加类别的层级
            this.categoryForm.catLevel = data.catLevel + 1;
            // 菜单的显示状态 1 显示 0 被删除
            this.categoryForm.showStatus = 1;
            // 排序 默认给 0
            this.categoryForm.sort = 0;

            // 重置更新的数据
            this.categoryForm.catId = null;
            this.categoryForm.name = "";
            this.categoryForm.icon = "";
            this.categoryForm.productUnit = "";
            // 更新新增或删除标志位为新增
            this.dialogType = true;
        },
        addDialog() {
            // 添加三级分类的类别信息
            this.$http({
                url: this.$http.adornUrl('/product/category/save'),
                method: 'post',
                data: this.$http.adornData(this.categoryForm, false)
            }).then(({ data }) => {
                if (data && data.code === 0) {
                    this.$message({
                        message: '数据添加成功',
                        type: 'success',
                    })
                    // 添加后关闭窗口
                    this.dialogVisible = false;
                    // 重新加载所有的菜单数据
                    this.getCategory();
                    // 设置默认展开的父节点信息
                    this.expandKeys = [this.categoryForm.parentCid]
                } else {
                    this.$message.error(data.msg)
                }
            })
        }, edit(data) {
            // 更新类别信息的方法
            // 更新新增或删除标志位为更新
            this.dialogType = false;
            // 获取最新的数据回写
            this.$http({
                url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
                method: 'post',
                data: this.$http.adornData(this.categoryForm, false)
            }).then(({ data }) => {
                console.log("获取的数据", data)
                // 表单数据回写
                this.categoryForm.name = data.category.name;
                this.categoryForm.productUnit = data.category.productUnit;
                this.categoryForm.icon = data.category.icon;
                this.categoryForm.catLevel = data.category.catLevel;
                this.categoryForm.parentCid = data.category.parentCid;
                // 填充更新数据的id
                this.categoryForm.catId = data.category.catId;
                this.categoryForm.showStatus = 1;
                this.categoryForm.sort = 0;
                // 打开更新的窗口
                this.dialogVisible = true;
            })
        }, editDialog(){
            // 添加三级分类的类别信息
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData(this.categoryForm, false)
            }).then(({ data }) => {
                if (data && data.code === 0) {
                    this.$message({
                        message: '更新数据成功',
                        type: 'success',
                    })
                    // 添加后关闭窗口
                    this.dialogVisible = false;
                    // 重新加载所有的菜单数据
                    this.getCategory();
                    // 设置默认展开的父节点信息
                    this.expandKeys = [this.categoryForm.parentCid]
                } else {
                    this.$message.error(data.msg)
                }
            })
        },
        submitForm() {
            console.log("dialogType",this.dialogType)
            // 判断当前操作是新增还是更新
            if(this.dialogType){
                // 添加操作
                this.addDialog();
            }else{
                // 更新操作
                this.editDialog();
            }
        },
        remove(node, data) {
            this.$confirm(`是否确认删除【${data.name}】分类?`, '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                // 传递的数据
                let ids = [data.catId];
                // 把删除的请求提交到后台服务
                this.$http({
                    url: this.$http.adornUrl('/product/category/delete'),
                    method: 'post',
                    data: this.$http.adornData(ids, false)
                }).then(({ data }) => {
                    if (data && data.code === 0) {
                        this.$message({
                            message: '删除成功',
                            type: 'success',
                        })
                        // 重新加载所有的菜单数据
                        this.getCategory();
                        // 设置默认展开的父节点信息
                        this.expandKeys = [node.parent.data.catId]
                    } else {
                        this.$message.error(data.msg)
                    }
                })
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });
            });
            console.log("删除", node, data);
        }
    },
    created() {
        this.getCategory();
    },
};
</script>

<style>
</style>