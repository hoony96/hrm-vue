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
        <el-table :data="sites" highlight-current-row v-loading="listLoading" style="width: 100%;">
            <el-table-column prop="name" label="名称" width="150" sortable>
            </el-table-column>
            <el-table-column prop="url" label="资源路径" width="250"  sortable>
            </el-table-column>
            <el-table-column prop="sn" label="标识" width="250"  sortable>
            </el-table-column>
            <el-table-column prop="description" label="描述" width="300"  sortable>
            </el-table-column>
            <el-table-column label="操作" min-width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
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
                <el-form-item label="域名" prop="email">
                    <el-input v-model="editForm.url" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="标识">
                    <el-input v-model="editForm.sn" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述" prop="age">
                    <el-input v-model.number="editForm.description" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--新增界面-->
        <el-dialog title="新增" :visible.sync="addFormVisible" :close-on-click-modal="false">
            <el-form :model="addForm" label-width="80px" ref="addForm">
                <el-form-item label="名称">
                    <el-input v-model="addForm.name" auto-complete="off" ></el-input>
                </el-form-item>
                <el-form-item label="域名" prop="email">
                    <el-input v-model="addForm.url" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="标识">
                    <el-input v-model="addForm.sn" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述" prop="age">
                    <el-input v-model.number="addForm.description" auto-complete="off"></el-input>
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

    export default {
        data() {
            return {
                sites: [],
                total: 0,
                page: 1,
                pageSize:10,
                listLoading: false,

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
                    url:'',
                    sn:'',
                    description:''
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
                    url:'',
                    sn:'',
                    description:''
                }

            }
        },
        methods: {
            //获取用户列表
            getsites() {
                let para = {
                    page: this.page,
                    rows: this.pageSize,
                };
                this.listLoading = true;
                this.$http.post("/page/site/page",para)
                    .then(res=>{
                        this.listLoading = false;
                        let { total,rows } = res.data;
                        this.total = total,
                        this.sites = rows;
                    })
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该数据吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = { id: row.id };
                    this.$http.delete("/page/site/"+row.id).then((res)=>{
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
                    url:'',
                    sn:'',
                    description:''
                };
            },
            //编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            let para = Object.assign({}, this.editForm);
                            this.$http.post("/page/site/save",para)
                                .then((res) => {
                                    this.editLoading = false;
                                    //NProgress.done();
                                    this.$message({
                                        message: '提交成功',
                                        type: 'success'
                                    });
                                    this.editForm =  {
                                        name:'',
                                        url:'',
                                        sn:'',
                                        description:''
                                    };
                                    this.editFormVisible = false;
                                    this.getsites();
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
                            this.$http.post("/page/site/save",para)
                                .then((res)=>{
                                    this.addLoading = false;
                                    let {success,message} = res.data;
                                    if(success){
                                        this.$message({
                                            message: message,
                                            type: 'success'
                                        });
                                    }
                                    this.addFormVisible = false;
                                    this.getsites();
                                })

                        });
                    }
                });
            }
        },
        mounted() {
            this.getsites();
        }
    }

</script>

<style scoped>

</style>