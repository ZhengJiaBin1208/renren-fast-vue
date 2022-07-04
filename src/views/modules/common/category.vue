<template>
    <div>
        <el-tree :data="data" :props="defaultProps"  node-key="catId"
        @node-click = "nodeclick"
        ></el-tree>
    </div>
</template>

<script>
/* eslint-disable */
export default {
    data() {
        return {
            data: [],
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
        }, 
        nodeclick(data,node,component){
            console.log("分类节点被点击了",data,node,component)
            this.$emit("show",data.catId)
        }
    },
    created() {
        this.getCategory();
    },
};
</script>

<style>
</style>