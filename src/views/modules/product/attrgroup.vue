<template>
  <el-row :gutter="20">
  <el-col :span="8">
  <category></category>
  </el-col>
  <el-col :span="16"><div class="grid-content bg-purple">表格</div></el-col>
</el-row>
</template>

<script>
import category from '../common/category.vue'
export default {
    data(){

    },components:{category}
}
</script>

<style>

</style>