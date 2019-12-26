<template id="courseType">
    <el-row style="height: 100%;border: 1px solid #DCDFE6;margin-top: 10px">
        <el-col :span="4" style="border-right: 1px solid #DCDFE6; min-height:620px;">
            <div class="tool-bar"></div>
            <div class="grid-content bg-purple">
                <el-tree :data="courseTypes" style="border:none;" :props="defaultProps"  @node-click="handleNodeClick">
                </el-tree>
            </div>
        </el-col>
        <el-col :span="17" style="margin-left: 10px;padding-top: 10px">

        </el-col>
    </el-row>
</template>
<style>
    .el-row {
        margin-bottom: 20px;
        height: 100%;
    }
    :last-child {
        margin-bottom: 0;
    }
    #courseType el-col {
        border: 1px solid red;
        border-radius: 4px;
    }
    .grid-content {
        border-radius: 4px;
        min-height: 36px;
    }
</style>

<script>
    export default {
        data() {
            return {
                courseTypes:[],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                }
            }
        },
        methods:{
            getTreeData(){
                // 发送一个异步请求: get请求 /product/courseType/treeData
                this.$http.get("/course/courseType/treeData").then(res=>{
                    console.log(this);
                    this.courseTypes = res.data;
                });

            }
        },
        mounted(){
            //对courseTypes数据赋值
           this.getTreeData();
        }
    };
</script>