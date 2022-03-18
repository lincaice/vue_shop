<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">
                首页
            </el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 添加搜索用户区 -->
            <el-row :gutter="20">
                <el-col :span="6">
                    <el-input
                        placeholder="请输入内容"
                        v-model="queryInfo.query"
                        clearable
                        @clear="getUserList(true)"
                    >
                        <el-button
                            slot="append"
                            icon="el-icon-search"
                            @click="getUserList(true)"
                        ></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true"
                        >添加用户</el-button
                    >
                </el-col>
            </el-row>
            <!-- 用户列表区 -->
            <el-table :data="userList" style="width: 100%" stripe border>
                <el-table-column type="index"> </el-table-column>
                <el-table-column prop="username" label="姓名">
                </el-table-column>
                <el-table-column prop="email" label="邮箱"></el-table-column>
                <el-table-column prop="mobile" label="电话"></el-table-column>
                <el-table-column prop="role_name" label="角色">
                </el-table-column>
                <el-table-column label="状态">
                    <template v-slot="scope">
                        <el-switch
                            v-model="scope.row.mg_state"
                            @change="userStateChanged(scope.row)"
                        ></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作">
                    <template v-slot="scope">
                        <!-- 修改按钮 -->
                        <el-button
                            type="primary"
                            size="mini"
                            icon="el-icon-edit"
                            @click="showEditDialog(scope.row.id)"
                        ></el-button>
                        <!-- 删除按钮 -->
                        <el-button
                            type="danger"
                            size="mini"
                            icon="el-icon-delete"
                            @click="removeUserById(scope.row.id)"
                        ></el-button>
                        <!-- 分配角色按钮 -->
                        <el-tooltip
                            effect="dark"
                            content="分配角色"
                            placement="top"
                            :enterable="false"
                        >
                            <el-button
                                type="warning"
                                size="mini"
                                icon="el-icon-setting"
                                @click="setRole(scope.row)"
                            ></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[1, 2, 4, 5]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total"
            >
            </el-pagination>
        </el-card>
        <!-- 添加用户对话框 -->
        <el-dialog
            title="添加用户"
            :visible.sync="addDialogVisible"
            width="50%"
            @close="addDialogClose"
        >
            <!-- 主体区域 -->
            <el-form
                ref="addFormRef"
                :model="addForm"
                :rules="addFormRules"
                label-width="70px"
                status-icon
            >
                <el-form-item label="用户名" prop="username">
                    <el-input v-model="addForm.username"></el-input>
                </el-form-item>
                <el-form-item label="密码" prop="password">
                    <el-input v-model="addForm.password"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="addForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser"> 确 定 </el-button>
            </span>
        </el-dialog>
        <!-- 修改用户对话框 -->
        <el-dialog
            title="修改用户"
            :visible.sync="editDialogVisible"
            width="50%"
            @close="editDialogClose"
        >
            <!-- 主体区域 -->
            <el-form
                ref="editFormRef"
                :model="editForm"
                :rules="editFormRules"
                label-width="70px"
            >
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUser"> 确 定 </el-button>
            </span>
        </el-dialog>
        <!-- 分配角色对话框 -->
        <el-dialog
            title="分配角色"
            :visible.sync="setRoleDialogVisible"
            width="30%"
            @close='setRoleDialogClosed'
        >
            <div>
                <p>当前用户是：{{ userInfo.username }}</p>
                <p>当前角色是：{{ userInfo.role_name }}</p>
                <p>
                    分配新角色：
                    <template>
                        <el-select
                            v-model="selectedRoleId"
                            filterable
                            placeholder="请选择需要分配的角色"
                        >
                            <el-option
                                v-for="item in roleList"
                                :key="item.id"
                                :label="item.roleName"
                                :value="item.id"
                            >
                            </el-option>
                        </el-select>
                    </template>
                </p>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRoleDialogVisible = false"
                    >取 消</el-button
                >
                <el-button type="primary" @click="saveRoleInfo">
                    确 定
                </el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: 'Users',
    data() {
        // 邮箱校验规则
        var checkEmail = (rule, value, callback) => {
            const regEmail =
                /^[A-Za-z\d]+([-_.][A-Za-z\d]+)*@([A-Za-z\d]+[-.])+[A-Za-z\d]{2,4}$/;
            if (regEmail.test(value)) {
                return callback();
            } else {
                return callback(new Error('请输入合法邮箱！'));
            }
        };
        // 密码校验规则
        var checkMobile = (rule, value, callback) => {
            const checkMobile = /^[1][3,4,5,7,8][0-9]{9}$/;
            if (checkMobile.test(value)) {
                return callback();
            } else {
                return callback(new Error('请输入正确的手机号！'));
            }
        };
        return {
            queryInfo: {
                query: '',
                pagenum: 1,
                pagesize: 2
            },
            userList: [],
            total: 0,
            // 控制添加用户对话框显示与隐藏
            addDialogVisible: false,
            // 控制修改用户对话框显示与隐藏
            editDialogVisible: false,
            // 添加用户表单数据
            addForm: {
                username: '',
                password: '',
                email: '',
                mobile: ''
            },
            // 添加表单验证规则对象
            addFormRules: {
                username: [
                    {
                        required: true,
                        message: '请输入用户名',
                        trigger: 'blur'
                    },
                    {
                        require: true,
                        min: 3,
                        max: 10,
                        message: '长度在 3 到 10 个字符',
                        trigger: 'blur'
                    }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    {
                        require: true,
                        min: 6,
                        max: 15,
                        message: '长度在 6 到 15 个字符',
                        trigger: 'blur'
                    }
                ],
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur' }
                ],
                mobile: [
                    { required: true, message: '请输入手机', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur' }
                ]
            },
            editForm: {},
            // 编辑表单验证规则对象
            editFormRules: {
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur' }
                ],
                mobile: [
                    { required: true, message: '请输入手机', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur' }
                ]
            },
            // 分配角色对话框显示与隐藏
            setRoleDialogVisible: false,
            // 需要被分配角色的用户信息
            userInfo: {},
            // 所有角色数据
            roleList: [],
            // 选中分配角色的id
            selectedRoleId: null
        };
    },
    methods: {
        // 获取用户列表
        async getUserList(isSearch) {
            if (isSearch === true) {
                this.queryInfo.pagenum = 1;
            }
            const { data: res } = await this.$http.get('users', {
                params: this.queryInfo
            });
            if (res.meta.status !== 200) {
                return this.$message.error('获取用户列表失败');
            }
            this.userList = res.data.users;
            this.total = res.data.total;
        },
        // 监听pagesize改变的事件（每页显示多少条数据）
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize;
            this.getUserList();
        },
        // 监听页码改变
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage;
            this.getUserList();
        },
        // 修改用户状态
        async userStateChanged(userInfo) {
            const { data: res } = await this.$http.put(
                `users/${userInfo.id}/state/${userInfo.mg_state}`
            );
            if (res.meta.status !== 200) {
                userInfo.mg_state = !userInfo.mg_state;
                this.$message.error('修改状态失败');
            }
            this.$message.success('修改状态成功');
        },
        // 添加对话框关闭事件
        addDialogClose() {
            this.$refs.addFormRef.resetFields();
        },
        // 编辑对话框关闭事件
        editDialogClose() {
            this.$refs.editFormRef.resetFields();
        },
        // 对话框内添加用户按钮确认事件
        addUser() {
            this.$refs.addFormRef.validate(async (valid) => {
                if (!valid) {
                    return;
                }
                const { data: res } = await this.$http.post(
                    'users',
                    this.addForm
                );
                if (res.meta.status !== 201) {
                    return this.$message.error('添加用户失败');
                }
                this.$message.success('添加用户成功');
                this.addDialogVisible = false;
                this.getUserList();
            });
        },
        // 修改用户按钮事件
        async showEditDialog(id) {
            const { data: res } = await this.$http.get('users/' + id);
            if (res.meta.status !== 200) {
                this.$message.error('查询用户信息失败');
            }
            this.editForm = res.data;
            this.editDialogVisible = true;
        },
        // 对话框内编辑用户按钮确认事件
        editUser() {
            this.$refs.editFormRef.validate(async (valid) => {
                if (!valid) {
                    return;
                }
                const { data: res } = await this.$http.put(
                    'users/' + this.editForm.id,
                    { email: this.editForm.email, mobile: this.editForm.mobile }
                );
                if (res.meta.status !== 200) {
                    this.$message.error('修改用户资料失败');
                }
                this.editDialogVisible = false;
                this.getUserList();
                this.$message.success('修改用户资料成功');
            });
        },
        // 删除用户按钮事件
        removeUserById(id) {
            this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
            })
                .then(async () => {
                    const { data: res } = await this.$http.delete(
                        'users/' + id
                    );
                    if (res.meta.status !== 200) {
                        this.$message({
                            type: 'error',
                            message: '删除失败!'
                        });
                    }
                    this.$message({
                        type: 'success',
                        message: '删除成功!'
                    });
                    this.getUserList();
                })
                .catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
        },
        // 展示分配角色对话框
        async setRole(userInfo) {
            const { data: res } = await this.$http.get('roles');
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败！');
            }
            this.roleList = res.data;
            this.userInfo = userInfo;
            this.setRoleDialogVisible = true;
        },
        // 点击按钮分配角色
        async saveRoleInfo(){
            if (! this.selectedRoleId) {
                return this.$message.error('请选择需要分配的角色');
            }
            const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`
                ,{ rid: this.selectedRoleId }
            );
            if (res.meta.status !== 200) {
                return this.$message.error('分配角色权限失败');
            }
            this.$message.success('分配角色权限成功');
            this.getUserList();
            this.setRoleDialogVisible = false;
        },
        // 关闭分配角色弹框
        setRoleDialogClosed(){
            this.userInfo = {};
            this.selectedRoleId = null;

        }
    },
    mounted() {
        this.getUserList();
    }
};
</script>

<style scoped lang='less'>
</style>