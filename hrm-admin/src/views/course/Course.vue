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
                    <!--<el-button type="primary" @click="handleMarket" :disabled="this.sels.length!=1">市场信息维护</el-button>-->
                    <!--<el-button type="primary" @click="handlePic" :disabled="this.sels.length!=1">图片信息维护</el-button>-->
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
            <el-table-column prop="name" label="名称" width="120" sortable>
            </el-table-column>
            <el-table-column prop="courseType.name" label="课程类型" width="100" sortable>
            </el-table-column>
            <el-table-column prop="tenantName" label="机构" width="100" sortable>
            </el-table-column>
            <el-table-column prop="userName" label="创建用户" width="120" sortable>
            </el-table-column>
            <el-table-column prop="startTime" label="上架时间" min-width="130" sortable :formatter="formatStartTime">
            </el-table-column>
            <el-table-column prop="endTime" label="下架时间" min-width="130" sortable :formatter="formatEndTime">
            </el-table-column>
            <el-table-column prop="status" label="课程状态" min-width="90" sortable>
                <template scope="scope">
                    <span v-if="scope.row.status == 1" style="color: green;">
                        上架
                    </span>
                    <span v-else style="color: red;">
                        未上架
                    </span>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="200">
                <template scope="scope">
                    <el-tooltip class="item" effect="light" content="编辑" placement="bottom">
                        <el-button size="small" @click="handleEdit(scope.$index, scope.row)" icon="el-icon-edit" circle></el-button>
                    </el-tooltip>
                    <el-tooltip class="item" effect="light" content="删除" placement="bottom">
                        <el-button size="small" @click="handleDel(scope.$index, scope.row)" icon="el-icon-delete" circle></el-button>
                    </el-tooltip>
                    <el-tooltip class="item" effect="light" content="市场信息维护" placement="bottom">
                        <el-button size="small" @click="handleMarket(scope.$index, scope.row)" icon="fa fa-shopping-basket" circle></el-button>
                    </el-tooltip>
                    <el-tooltip class="item" effect="light" content="资源信息维护" placement="bottom">
                        <el-button  size="small" @click="handlePic(scope.$index, scope.row)" icon="fa fa-object-group" circle></el-button>
                    </el-tooltip>
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
                <el-form-item label="收费规则">
                    <el-radio-group v-model="market.charge">
                        <el-radio :label="5">付费</el-radio>
                        <el-radio :label="6">免费</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="有效性">
                    <el-radio-group v-model="market.valid">
                        <el-radio :label="9">有效</el-radio>
                        <el-radio :label="10">失效</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="失效时间">
                    <el-date-picker
                            v-model="market.expires"
                            type="datetime"
                            placeholder="请选择失效时间">
                    </el-date-picker>
                </el-form-item>
                <el-form-item label="联系方式">
                    <el-input v-model="market.qq" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="价格">
                    <el-input v-model="market.price" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="原价">
                    <el-input v-model="market.priceOld" auto-complete="off"></el-input>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="marketFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="handleMarketSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>


        <!--  维护图片信息 -->
        <el-dialog title="图片信息维护" :visible.sync="picForm" :close-on-click-modal="false">
            <el-form :model="pics" label-width="130px" ref="pics">
                <el-upload
                        class="upload-demo"
                        action="http://localhost:9527/services/file/file/upload"
                        :before-remove="handleRemove"
                        :on-success="handleSuccess"
                        :file-list="fileList"
                        limit="1"
                        list-type="picture">
                    <el-button size="small" type="primary">点击上传</el-button>
                    <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过1024kb</div>
                </el-upload>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="picForm = false">取消</el-button>
                <el-button type="primary" @click.native="handlePicSubmit" :loading="addLoading">提交</el-button>
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
                //  图片信息维护
                picForm:false,
                pics:{
                    courseId:'',
                    resources:''
                },
                fileList:[],

                // 富文本框内容
                intro:'',
                content:'',

                //  市场信息维护
                marketFormVisible:false,
                market:{
                    charge:5,
                    valid:9,
                    expires:'',
                    qq:'',
                    price:'',
                    priceOld:'',
                    courseId:''
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
                param.typeId = sessionStorage.getItem("typeId");
                sessionStorage.removeItem("typeId");
                this.$http.post("/course/course/page",param)
                    .then(res=>{
                        let {total,rows} = res.data;
                        this.total = total;
                        this.courses = rows;
                    })
            },

            // 处理新增课程
            handleAdd(){
                this.$refs.addCourse.resetFields();
                this.addCourseVisible = true;
                this.description = '';
                this.intro = '';
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
                            this.$refs.editCourse.resetFields();
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
            handleMarket(index,row){
                this.marketFormVisible = true;
                this.market.courseId = row.id;
                this.$http.get("/course/courseMarket/"+this.market.courseId)
                    .then(res=>{
                        this.market = res.data;
                    })
            },
            handleMarketSubmit(){
                let param = Object.assign({}, this.market);
                param.expires = new Date(param.expires).getTime();
                this.$http.post("/course/courseMarket/update",param)
                    .then(res=>{
                        let {success,message} = res.data;
                        if (success){
                            this.$message({
                                message: message,
                                type: 'success'
                            });
                            this.$refs.market.resetFields();
                            this.marketFormVisible = false;
                        }else{
                            this.$message({
                                message: message,
                                type: 'error'
                            });
                        }
                    })
            },


            // 处理 资源图片信息
            handlePic(index,row){
                this.picForm = true;
                this.pics.courseId = row.id;
                this.$http.post("/course/courseResource/list")
                    .then(res=>{
                        this.fileList = res.data;
                    })
            },
            handlePicSubmit(){
                let param = Object.assign({}, this.pics);
                this.$http.post("/course/courseResource/save",param)
                    .then(res=>{
                        let {success,message} = res.data;
                        if (success){
                            this.$message({
                                message: message,
                                type: 'success'
                            });
                            this.$refs.pics.resetFields();
                            this.picForm = false;
                        }else{
                            this.$message({
                                message: message,
                                type: 'error'
                            });
                        }
                    })
            },

            // 处理logo上传前的删除行为
            handleRemove(file) {
                this.file_id = file.response.resultObj;
                setTimeout(()=>{
                    this.$http.get("/file/file/delete?file_id="+this.file_id)
                        .then(res=>{
                            let{ success,message } = res.data;
                            if(!success){
                                this.$message({
                                    message: message ,
                                    type: 'error'
                                });
                                return false;
                            }
                        })
                },500)
            },
            // 处理上传之后接收 返回数据
            handleSuccess(response){
                let{ success,message,resultObj } = response;
                this.pics.resources = "http://172.16.5.137:22122/"+resultObj;
                if(!success){
                    this.$message({
                        message: message ,
                        type: 'error'
                    });
                }
            },


            // 处理翻页
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
            },
        },
        mounted(){
            this.getTreeData();
            this.getCourses();
        }
    }
</script>

<style scoped>
    #edit span{
        display: none;
    }
    #edit:hover span{
        display: initial;
    }

</style>