<template xmlns:v-quill="http://www.w3.org/1999/xhtml">
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <!--<el-input v-model="filters.keyword" placeholder="关键字"></el-input>-->
                    <el-select v-model="filters.status" clearable placeholder="课程状态">
                        <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getCourses">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增课程</el-button>
                    <el-button type="primary" @click="handleMarket">市场信息</el-button>
                    <el-button type="primary" @click="handlePic">图片信息</el-button>
                    <el-form-item>
                        <el-button type="primary" @click="online" :disabled="this.sels.length===0">上线</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="offline" :disabled="this.sels.length===0">下线</el-button>
                    </el-form-item>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="courses" highlight-current-row @selection-change="selsChange" style="width: 100%;">

            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
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
            <el-table-column prop="status" label="课程状态" min-width="180" sortable>
                <template scope="scope">
                    <span v-if="scope.row.status == 1" style="color: green;">
                        上架
                    </span>
                    <span v-else style="color: red;">
                        未上架
                    </span>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
            </el-pagination>
        </el-col>


        <!--新增界面-->
        <el-dialog title="新增课程" :visible.sync="addCourseVisible" :close-on-click-modal="false">
            <el-form :model="addCourse" label-width="130px" :rules="addCourseRules" ref="addCourse">
                <el-form-item label="名称">
                    <el-input v-model="addCourse.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="适用人群">
                    <el-input v-model="addCourse.users" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="所属课程类型">
                    <select-tree
                            :props="props"
                            :options="courseTypes"
                            :value="valueId"
                            :clearable="true"
                            :accordion="true"
                            @getValue="getValue($event)"
                    />
                </el-form-item>
                <el-form-item label="难度级别">
                    <el-radio-group v-model="addCourse.grade">
                        <el-radio :label="1">黑铁</el-radio>
                        <el-radio :label="2">青铜</el-radio>
                        <el-radio :label="3">白银</el-radio>
                        <el-radio :label="4">黄金</el-radio>
                    </el-radio-group>
                </el-form-item>

                <el-form-item label="课程简介">
                    <el-input v-model="intro" auto-complete="off"></el-input>
                </el-form-item>

                <el-form-item label="课程详情">
                    <quill-editor v-model="content"
                                  ref="myQuillEditor"
                                  :options="editorOption"
                                  @blur="onEditorBlur($event)">
                    </quill-editor>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addCourseVisible = false">取消</el-button>
                <el-button type="primary" @click.native="handleAddSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>


        <!--编辑界面-->
        <el-dialog title="编辑课程" :visible.sync="editCourseVisible" :close-on-click-modal="false">
            <el-form :model="editCourse" label-width="130px" :rules="editCourseRules" ref="editCourse">
                <el-form-item label="名称">
                    <el-input v-model="editCourse.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="适用人群">
                    <el-input v-model="editCourse.users" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="所属课程类型">
                    <select-tree
                            :props="props"
                            :options="courseTypes"
                            :value="valueId"
                            :clearable="true"
                            :accordion="true"
                            @getValue="getValue($event)"
                    />
                </el-form-item>
                <el-form-item label="难度级别">
                    <el-radio-group v-model="editCourse.grade">
                        <el-radio :label="1">黑铁</el-radio>
                        <el-radio :label="2">青铜</el-radio>
                        <el-radio :label="3">白银</el-radio>
                        <el-radio :label="4">黄金</el-radio>
                    </el-radio-group>
                </el-form-item>

                <el-form-item label="课程简介">
                    <el-input v-model="intro" auto-complete="off"></el-input>
                </el-form-item>

                <el-form-item label="课程详情">
                    <quill-editor v-model="content"
                                  ref="myQuillEditor"
                                  :options="editorOption"
                                  @blur="onEditorBlur($event)">
                    </quill-editor>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editCourseVisible = false">取消</el-button>
                <el-button type="primary" @click.native="handleEditSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--  维护市场信息 -->
        <el-dialog title="市场信息维护" :visible.sync="marketFormVisible" :close-on-click-modal="false">
            <el-form :model="market" label-width="130px" ref="market">
                <el-form-item label="名称">
                    <el-input v-model="market.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="适用人群">
                    <el-input v-model="market.users" auto-complete="off"></el-input>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editCourseVisible = false">取消</el-button>
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
        name: "course",
        data(){
            return {
                // 富文本框内容
                intro:'',
                content:'',
                //  市场信息维护
                marketFormVisible:false,
                market:{
                    charge:'',
                    valid:'',
                    qq:'',
                    price:'',
                    price_old:'',
                    course_id:''
                },

                // 编辑所需
                editCourseVisible:false,
                editCourse:{
                    id:'',
                    name:'',
                    users:'',
                    courseTypeId:'',
                    grade:''
                },

                // 课程下拉数
                props:{
                    // 配置项（必选）
                    value: "id",
                    label: "name",
                    children: "children"
                },
                valueId:'',
                courseTypes:[],

                options:[
                    {
                        value:0,
                        label:"未上架"
                    },
                    {
                        value:1,
                        label:"上架"
                    }
                ],
                filters:{
                    keyword:"",
                    status:null
                },
                courses:[],
                sels:[],//多选框选中的行
                pageNum:1,
                pageSize:10,
                total:0,
                addCourseVisible:false,
                addCourse:{
                    name:'',
                    users:'',
                    courseTypeId:'',
                    grade:1
                },
                addCourseRules:[]
            }
        },
        methods:{
            // 下拉树
            // 获取下拉框节点的id
            getValue(value){
                this.valueId = value;
            },
            //  获取 下拉树信息
            getTreeData(){
                // 发送一个异步请求: get请求 /product/courseType/treeData
                this.$http.get("/course/courseType/treeData").then(res=>{
                    this.courseTypes = res.data;
                });

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
                param.keyword = this.filters.keyword;
                param.status = this.filters.status;
                this.$http.post("/course/course/page",param)
                    .then(res=>{
                        let {total,rows} = res.data;
                        this.total = total;
                        this.courses = rows;
                    })
            },

            // 处理新增课程
            handleAdd(){
                this.addCourseVisible = true;
            },
            handleAddSubmit(){
                let param = {};
                param = Object.assign({}, this.addCourse);
                param.courseTypeId = this.valueId;
                param.description = this.content;
                param.intro = this.intro;
                this.$http.post("/course/course/insert",param)
                    .then(res=>{
                        let {success,message} = res.data;
                        if(success){
                            this.$message({
                                type : "success",
                                message : message
                            })
                            this.addCourseVisible = false;
                            this.$refs.addCourse.resetFields()
                            ;
                        }else{
                            this.$message({
                                type : "error",
                                message : message
                            })
                        }
                    })
            },

            // 处理 编辑课程
            handleEdit(index,row){
                this.editCourse = row;
                this.valueId = row.courseTypeId;
                this.$http.get("/course/courseDetail/"+row.id)
                    .then(res=>{
                        this.content = res.data.description;
                        this.intro = res.data.intro;
                    })
                this.editCourseVisible = true;

            },
            handleEditSubmit(){
                let param = {};
                param = Object.assign({}, this.editCourse);
                param.courseTypeId = this.valueId;
                param.description = this.content;
                param.intro = this.intro;
                this.$http.post("/course/course/insert",param)
                    .then(res=>{
                        let {success,message} = res.data;
                        if(success){
                            this.$message({
                                type : "success",
                                message : message
                            })
                            this.editCourseVisible = false;
                            this.editCourse.resetField();
                        }else{
                            this.$message({
                                type : "error",
                                message : message
                            })
                        }
                    })
            },


            // 处理删除
            handleDel(index,row){
                this.$http.delete("/course/course/"+row.id)
                    .then(res=>{
                        let {success,message} = res.data;
                        if(success){
                            this.$message({
                                type : "success",
                                message : message
                            })
                            this.getCourses();
                        }else{
                            this.$message({
                                type : "error",
                                message : message
                            })
                        }
                    })
            },
            // 处理批量删除
            selsChange(selection){
                this.sels = selection;
            },
            batchRemove(){
                let ids = '';
                for(let i=0;i < this.sels.length;i++){
                    ids += this.sels[i].id;
                    ids += ","
                }
                // let ids = this.sels.map(e=>e.id);  直接转为 List ids
                this.$confirm('确认删除该数据吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.get("/course/course/batchRemove?ids=" + ids)
                        .then(res => {
                            let {success, message} = res.data;
                            if (success) {
                                this.$message({
                                    type: "success",
                                    message: message
                                })
                                this.getCourses();
                            } else {
                                this.$message({
                                    type: "error",
                                    message: message
                                })
                            }
                        })
                })
            },

            // 处理修改市场 信息
            handleMarket(){
                this.marketFormVisible = true;
            },

            // 处理 资源图片信息
            handlePic(){

            },
            handleCurrentChange(val){
                this.pageNum = val;
                this.getCourses();
            },

            // 处理上线
            online(){
                let param = this.sels.map(e => e.id);//es6语法
                this.$http.post("/course/course/online",param) //$.Post(.....)
                    .then(result=>{
                        let {success,message} = result.data;
                        if (success){
                            this.$message({
                                message: '操作成功!',
                                type: 'success'
                            });
                            //刷新数据
                            this.getCourses();
                        }else{
                            this.$message({
                                message: '操作失败!',
                                type: 'error'
                            });
                        }

                    });
            },

            // 处理下线
            offline(){
                let param = this.sels.map(e => e.id);//es6语法
                this.$http.post("/course/course/offline",param) //$.Post(.....)
                    .then(result=>{
                        let {success,message} = result.data;
                        if (success){
                            this.$message({
                                message: '操作成功!',
                                type: 'success'
                            });
                            //刷新数据
                            this.getCourses();
                        }else{
                            this.$message({
                                message: '操作失败!',
                                type: 'error'
                            });
                        }

                    });
            }
        },
        mounted(){
            this.getCourses();
            this.getTreeData();
        }
    }
</script>

<style scoped>

</style>