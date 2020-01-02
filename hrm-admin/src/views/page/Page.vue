<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true">
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="pages" highlight-current-row v-loading="listLoading" style="width: 100%;">
            <el-table-column prop="name" label="名称" width="80" sortable>
        </el-table-column>
            <el-table-column prop="alias" label="别名" width="80" sortable>
            </el-table-column>
            <el-table-column prop="type" label="类型" width="80" sortable>
                <template scope="scope">
                    <span v-if="scope.row.type==1">静态</span>
                    <span v-else>动态</span>
                </template>
            </el-table-column>
            <el-table-column prop="physicalPath" label="物理路径" width="180" sortable>
            </el-table-column>
            <el-table-column prop="createTime" label="创建时间" width="100" :formatter="formatCreateTime" sortable>
            </el-table-column>
            <el-table-column prop="siteName" label="对应站点" width="120"  sortable>
            </el-table-column>
            <el-table-column prop="templateUrl" label="模板路径" width="150"  sortable>
            </el-table-column>
            <el-table-column prop="pageConfig.dataKey" label="模板数据" width="150"  sortable>
            </el-table-column>
            <el-table-column prop="pageConfig.pageUrl" label="页面路径" width="100"  sortable>
            </el-table-column>
            <el-table-column label="操作" min-width="150">
                <template scope="scope">
                    <el-button type="danger" size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--编辑界面-->
        <el-dialog title="编辑" :visible.sync="editFormVisible" :close-on-click-modal="false">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="名称">
                    <el-input v-model="editForm.name" auto-complete="off" ></el-input>
                </el-form-item>
                <el-form-item label="别名">
                    <el-input v-model="editForm.alias" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="存储路径">
                    <el-input v-model="editForm.physicalPath" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类型">
                    <el-radio-group v-model="editForm.type" style="width:100%">
                        <el-radio :label="1">静态</el-radio>
                        <el-radio :label="2">动态</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="所属站点">
                    <el-select v-model="editForm.siteId" clearable placeholder="请选择所属站点" style="width:100%">
                        <el-option
                                v-for="item in sites"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="模板地址">
                    <el-input v-model="editForm.templateUrl" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--新增界面-->
        <el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="100px" :rules="addFormRules" ref="addForm">
                <el-form-item label="名称">
                <el-input v-model="addForm.name" auto-complete="off" ></el-input>
            </el-form-item>
                <el-form-item label="别名">
                    <el-input v-model="addForm.alias" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="存储路径">
                    <el-input v-model="addForm.physicalPath" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类型">
                    <el-radio-group v-model="addForm.type" style="width:100%">
                        <el-radio :label="1">静态</el-radio>
                        <el-radio :label="2">动态</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="所属站点">
                    <el-select v-model="addForm.siteId" clearable placeholder="请选择所属站点" style="width:100%">
                        <el-option
                                v-for="item in sites"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="模板地址">
                    <el-input v-model="addForm.templateUrl" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    import util from '../../common/js/util'
    //import NProgress from 'nprogress'
    import { getUserListPage, removeUser, batchRemoveUser, editUser, addUser } from '../../api/api';

    export default {
        data() {
            return {
                // 站点
                sites:[],
                pages: [],
                total: 0,
                page: 1,
                pageSize:10,
                listLoading: false,
                sels: [],//列表选中列

                editFormVisible: false,//编辑界面是否显示
                editLoading: false,
                editFormRules: {
                    username: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ],
                    email:[
                        { type:"email",message:"请输入正确的邮箱", trigger: 'blur'}
                    ],
                    age:[
                        {type:"integer",min:18,max:65,message:"请输入正确的年龄", trigger: 'blur'}
                    ]
                },
                //编辑界面数据
                editForm: {
                    name:'',
                    alias:'',
                    physicalPath:'',
                    type:'',
                    siteId:'',
                    templateUrl:''
                },

                addFormVisible: false,//新增界面是否显示
                addLoading: false,
                addFormRules: {
                    username: [
                        { required: true, message: '请输入姓名', trigger: 'blur' }
                    ],
                    email:[
                        { type:"email",message:"请输入正确的邮箱", trigger: 'blur'}
                    ],
                    age:[
                        {type:"integer",min:18,max:65,message:"请输入正确的年龄", trigger: 'blur'}
                    ]
                },
                //新增界面数据
                addForm: {
                    name:'',
                    alias:'',
                    physicalPath:'',
                    type:'',
                    siteId:'',
                    templateUrl:''
                }

            }
        },
        methods: {
            // 处理时间
            formatCreateTime(row){
               return this.formatTime(row.createTime);
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
                let timeStr = year+"-"+this.formatNum(month)+"-"+this.formatNum(day)
                return timeStr
            },
            formatNum(num){
                if (num<10){
                    return "0"+num;
                }
                return num;
            },

            // 获取下拉站点
            getSite(){
                this.$http.get("/page/site/list")
                    .then(res=>{
                        this.sites = res.data;
                    })
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getpages();
            },
            //获取页面列表
            getpages() {
                let para = {
                    page: this.page,
                    rows: this.pageSize,
                };
                // console.log(this.keyword)
                this.listLoading = true;
                //NProgress.start();
                this.$http.post("/page/pager/page",para)
                    .then(res=>{
                        this.listLoading = false;
                        let { total,rows } = res.data;
                        this.total = total,
                        this.pages = rows;
                    })
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该数据吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    let para = { id: row.id };
                    this.$http.get("/delete?id="+row.id).then((res)=>{
                        this.listLoading = false;
                        let {success,message} = res.data;
                        if(success){
                            this.$message({
                                message: message,
                                type: 'success'
                            });
                        }
                    })
                }).catch(() => {

                });
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.editFormVisible = true;
                this.editForm = Object.assign({}, row);
            },
            //显示新增界面
            handleAdd: function () {
                this.addFormVisible = true;
                this.addForm = {
                    name:'',
                    alias:'',
                    physicalPath:'',
                    type:'',
                    siteId:'',
                    templateUrl:''
                };
            },
            //编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            //NProgress.start();
                            let para = Object.assign({}, this.editForm);
                            this.$http.post("/page/pager/save",para)
                                .then(res => {
                                    let {success,message} = res.data;
                                    if(success){
                                        this.editLoading = false;
                                        this.$message({
                                            message: message,
                                            type: 'success'
                                        });
                                        this.$refs['editForm'].resetFields();
                                        this.editFormVisible = false;
                                        this.getpages();
                                    }
                            });
                        });
                    }
                });
            },
            //新增
            addSubmit: function () {
                this.$refs.addForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.addLoading = true;
                            let para = Object.assign({}, this.addForm);
                            this.$http.post("/page/pager/save",para)
                                .then((res)=>{
                                    this.addLoading = false;
                                    let {success,message} = res.data;
                                    if(success){
                                        this.$message({
                                            message: message,
                                            type: 'success'
                                        });
                                    }
                                    this.getpages();
                                    this.addFormVisible = false;
                                })

                        });
                    }
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            }
        },
        mounted() {
            this.getpages();
            this.getSite();
        }
    }

</script>

<style scoped>
    .el-dialog{
        width: 40% !important;
    }
</style>