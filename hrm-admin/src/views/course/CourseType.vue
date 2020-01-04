<template id="courseType">
    <section>
        <el-row style="height: 100%;border: 1px solid #DCDFE6;margin-top: 10px">
            <el-col :span="5" style="border-right: 1px solid #DCDFE6; min-height:600px;">
                <div class="tool-bar">
                    <span class="el-icon-info"> 单击右键,进行新增/修改/删除操作</span>
                    <!--<el-button type="primary" class="el-icon-plus" circle></el-button>
                    <el-button type="success" class="el-icon-edit" circle></el-button>
                    <el-button type="info" class="el-icon-delete" circle></el-button>-->
                </div>
                <hr/>
                <div class="grid-content bg-purple">
                    <el-tree :data="courseTypes" style="border:none;" :props="defaultProps"
                             @node-contextmenu='rightClick' @node-click="handleNodeClick">
                    </el-tree>
                </div>
                <div id="perTreeMenu" v-if="divShow" class="tree_menu" :style="{...rightMenu}">
                    <ul>
                        <li><el-button type="primary" size="mini" class="el-icon-plus" @click="handleAdd"> 新增</el-button></li>
                        <li><el-button type="success" size="mini" class="el-icon-edit" @click="handleEdit"> 编辑</el-button></li>
                        <li><el-button type="info" size="mini" class="el-icon-delete" @click="handleDelete"> 删除</el-button></li>
                    </ul>
                </div>
            </el-col>
            <el-col :span="18" style="margin-left: 10px;padding-top: 10px" >
                <el-table :data="courses" highlight-current-row style="width: 100%;">
                    <el-table-column prop="name" label="名称" width="120" sortable>
                    </el-table-column>
                    <el-table-column prop="courseType.name" label="课程类型" width="100" sortable>
                    </el-table-column>
                    <el-table-column prop="tenantName" label="机构" width="100" sortable>
                    </el-table-column>
                    <el-table-column prop="userName" label="创建用户" width="120" sortable>
                    </el-table-column>
                    <el-table-column prop="startTime" label="上架时间" min-width="180" sortable :formatter="formatStartTime">
                    </el-table-column>
                    <el-table-column prop="endTime" label="下架时间" min-width="180" sortable :formatter="formatEndTime">
                    </el-table-column>
                    <el-table-column prop="status" label="课程状态" min-width="100" sortable>
                        <template scope="scope">
                    <span v-if="scope.row.status == 1" style="color: green;">
                        上架
                    </span>
                            <span v-else style="color: red;">
                        未上架
                    </span>
                        </template>
                    </el-table-column>
                </el-table>
                <!--工具条-->
                <el-col :span="24" class="toolbar">
                    <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
                    </el-pagination>
                </el-col>
            </el-col>
        </el-row>

        <!--新增界面-->
        <el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" ref="addForm">
                <el-form-item label="目录名称">
                    <el-input v-model="addForm.name" auto-complete="off" ></el-input>
                </el-form-item>
                <el-form-item label="父级目录">
                    <select-tree
                        :props="props"
                        :options="courseTypes"
                        :value="valueId"
                        :clearable="true"
                        :accordion="true"
                        @getValue="getValue($event)"
                />
                </el-form-item>
                <el-form-item label="logo">
                    <el-input v-model="addForm.logo" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model.number="addForm.description" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="总个数">
                    <el-input v-model="addForm.totalCount" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="handleAddSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>


        <!--编辑界面-->
        <el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" ref="editForm">
                <el-form-item label="目录名称">
                    <el-input v-model="editForm.name" auto-complete="off" ></el-input>
                </el-form-item>
                <el-form-item label="父级目录">
                    <select-tree
                            :props="props"
                            :options="courseTypes"
                            :value="valueId"
                            :clearable="true"
                            :accordion="true"
                            @getValue="getValue($event)"
                    />
                </el-form-item>
                <el-form-item label="logo">
                    <el-input v-model="editForm.logo" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model.number="editForm.description" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="总个数">
                    <el-input v-model="editForm.totalCount" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="handleEditSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>


    </section>
</template>

<script>
    import SelectTree from '@/treeSelector/SelectTree.vue'
    export default {
        components:{
            SelectTree
        },
        data() {
            return {
                // 右侧表格展示
                courses:[],
                pageNum:1,
                pageSize:10,
                total:0,


                addLoading:false,
                editFormVisible:false,
                props:{
                    // 配置项（必选）
                    value: "id",
                    label: "name",
                    children: "children"
                },
                selectedContent:[],
                addFormVisible:false,
                divShow:false,
                courseTypes:[],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                addForm:{
                    "pid":'',
                    "name":"",
                    "logo":'',
                    "description":'',
                    "totalCount":0
                },
                editForm:{
                    "id":'',
                    "pid":'',
                    "name":"",
                    "logo":'',
                    "description":'',
                    "totalCount":0
                },
                // 一级目录
                parents:[],
                // 下拉框 选择父级目录的id
                valueId:'',

                // 删除 传入节点的id
                contentId:''
            }
        },
        methods:{
            handleCurrentChange(val){
                this.pageNum = val;
                this.getCourses();
            },
            formatStartTime(row, column){
                return this.formatTime(row.startTime);
            },
            formatEndTime(row, column){
                return this.formatTime(row.endTime);
            },
            formatTime(time){
                if (!time){
                    return "";
                }
                let date = new Date(time);
                let year = date.getFullYear();
                let month = date.getMonth()+1;
                let day = date.getDay();
                let hours = date.getHours();
                let minutes = date.getMinutes();
                let seconds = date.getSeconds();
                let timeStr = year+"-"+this.formatNum(month)+"-"+this.formatNum(day)+" "
                    + this.formatNum(hours)+":"+this.formatNum(minutes)+":"+this.formatNum(seconds)
                return timeStr
            },
            formatNum(num){
                if (num<10){
                    return "0"+num;
                }
                return num;
            },

            //获取课程
            getCourses(){
                let param = {};
                param.page = this.pageNum;
                param.rows = this.pageSize;
                param.typeId = this.contentId;
                this.$http.post("/course/course/page",param)
                    .then(res=>{
                        let {total,rows} = res.data;
                        this.total = total;
                        this.courses = rows;
                    })
            },

            // 处理删除
            handleDelete(){
                this.$confirm('确认删除该数据吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.delete("/course/courseType/"+this.contentId).then((res)=>{
                        this.listLoading = false;
                        let {success,message} = res.data;
                        if(success){
                            this.$message({
                                message: message,
                                type: 'success'
                            });
                            this.getTreeData();
                        }
                    })
                }).catch(() => {

                });
            },
            // 获取下拉框节点的id
            getValue(value){
                this.valueId = value;
            },
            // 编辑目录
            handleEdit(){
                this.$http.get("/course/courseType/"+this.contentId)
                    .then(res=>{
                        this.editForm = res.data;
                    })
                setTimeout(()=>{
                    this.editFormVisible = true
                },500)
            },
            handleEditSubmit(){
                this.editFormVisible = false;
                this.editForm.pid = this.valueId;
                this.editForm.id = this.contentId;
                let para = Object.assign({}, this.editForm);
                this.$http.post("/course/courseType/save",para)
                    .then(res=>{
                        let{success,message} = res.data;
                        if(success){
                            this.$message({
                                message:message,
                                type: 'success'
                            })
                        }
                        this.getTreeData();
                        this.valueId = '';
                    })
            },
            // 新增目录
            handleAdd(){
                this.addFormVisible = true;
            },
            handleAddSubmit(){
                this.addFormVisible = false;
                this.addForm.pid = this.valueId;
                let para = Object.assign({}, this.addForm);
                this.$http.post("/course/courseType/save",para)
                    .then(res=>{
                        let{success,message} = res.data;
                        if(success){
                            this.$message({
                                message:message,
                                type: 'success'
                            })
                        }
                        this.addFormVisible = false;
                        this.getTreeData();
                        this.addForm.resetFields();
                    })
            },
            // 获取左侧目录的所有数据
            getTreeData(){
                // 发送一个异步请求: get请求 /product/courseType/treeData
                this.$http.get("/course/courseType/treeData").then(res=>{
                    this.courseTypes = res.data;
                    // debugger;
                    for(let i=0;i<this.courseTypes.length;i++){
                        if(this.courseTypes[i].children.length > 0){
                            this.parents.push(this.courseTypes[i]);
                        }
                    }
                });

            },
            // 左键显示 课程细节
            handleNodeClick(data){
                // data 是当前节点的数据
                if(data.children.length == 0){
                    this.contentId = data.id;
                    this.getCourses();
                }

            },
            // 右键菜单显示
            rightClick(e,data,node,comp){
                this.valueId = data.pid;
                this.contentId = data.id;
                // debugger;
                this.rightMenu = {top:e.pageY+'px',left:e.pageX+'px'};
                this.divShow = true;
                const self = this;
                document.onclick=function(ev){
                    if(ev.target!==document.getElementById('perTreeMenu')){
                        self.divShow = false
                    }
                }
            }
        },
        mounted(){
            //对courseTypes数据赋值
            this.getTreeData();
            window.oncontextmenu = function(){return false;}
        }
    };
</script>

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
    .tool-bar{
        height: 30px;
    }
    .tool-bar span{
        line-height: 50px;
        color: #15ac86;
        margin-left: 40px;
        font-weight: bold;
        font-size: small;
    }

    .tree_menu {
        position: fixed;
        display: block;
        z-index: 20000;
        background-color: #fff;
        padding: 5px 0;
        border: 1px solid #ebeef5;
        border-radius: 4px;
        box-shadow: 0 2px 12px 0 rgba(0, 0, 0, .1);
    }
    .tree_menu ul {
        margin: 0;
        padding: 0;
        width: 80px;
    }
    .tree_menu ul li {
        list-style: none;
        padding: 0 15px;
        font-size: 14px;
        line-height: 30px;
        cursor: pointer;
    }
    .tree_menu ul li button{
        margin-left: -10px;
    }
    .tree_menu ul li:hover {
        background-color: #ebeef5
    }
</style>
