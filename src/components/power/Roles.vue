<template>
    <div>
        <!-- 面包屑导航区域 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">
                首页
            </el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <!-- 添加角色按钮区域 -->
            <el-row :gutter="10">
                <el-col :span="6">
                    <el-button type="primary" @click="addRole">
                        添加角色
                    </el-button>
                </el-col>
            </el-row>

            <!-- 角色列表区域 -->
            <el-table style="width: 100%" :data="roleList" border stripe>
                <el-table-column type="expand">
                    <template v-slot="scope">
                        <el-row
                            class="bdbottom vcenter"
                            :gutter="10"
                            v-for="item1 in scope.row.children"
                            :key="item1.id"
                        >
                            <!-- 渲染一级权限 -->
                            <el-col :span="4">
                                <el-tag
                                    closable
                                    @close="
                                        removeRightById(scope.row, item1.id)
                                    "
                                    >{{ item1.authName }}</el-tag
                                >
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二/三级权限 -->
                            <el-col :span="20">
                                <el-row
                                    class="bdtbottom vcenter"
                                    v-for="item2 in item1.children"
                                    :key="item2.id"
                                >
                                    <el-col :span="6">
                                        <el-tag
                                            type="success"
                                            closable
                                            @close="
                                                removeRightById(
                                                    scope.row,
                                                    item2.id
                                                )
                                            "
                                        >
                                            {{ item2.authName }}
                                        </el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag
                                            type="warning"
                                            v-for="item3 in item2.children"
                                            :key="item3.id"
                                            closable
                                            @close="
                                                removeRightById(
                                                    scope.row,
                                                    item3.id
                                                )
                                            "
                                        >
                                            {{ item3.authName }}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type="index"> </el-table-column>
                <el-table-column
                    label="角色名称"
                    prop="roleName"
                ></el-table-column>
                <el-table-column
                    label="角色描述"
                    prop="roleDesc"
                ></el-table-column>
                <el-table-column label="操作">
                    <template v-slot="scope">
                        <el-button
                            type="primary"
                            size="small"
                            icon="el-icon-edit"
                            @click="updateRoleById(scope.row)"
                        >
                            编辑
                        </el-button>
                        <el-button
                            type="danger"
                            size="small"
                            icon="el-icon-delete"
                            @click="removeRoleById(scope.row.id)"
                        >
                            删除
                        </el-button>
                        <el-button
                            type="warning"
                            size="small"
                            icon="el-icon-setting"
                            @click="showSetRightDialog(scope.row)"
                        >
                            分配权限
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!-- 添加角色视图 -->
        <el-dialog
            title="添加角色"
            :visible.sync="addRolesVisible"
            width="50%"
            @close="addRoleClose"
        >
            <!-- 主体区域 -->
            <el-form
                ref="addRolesFormRef"
                :model="addRolesForm"
                :rules="addRolesFormRules"
                label-width="70px"
                status-icon
            >
                <el-form-item label="角色名" prop="roleName">
                    <el-input v-model="addRolesForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                    <el-input v-model="addRolesForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addRolesVisible = false">取 消</el-button>
                <el-button type="primary" @click="addOrUpdateRole">
                    确 定
                </el-button>
            </span>
        </el-dialog>
        <!-- 分配权限对话框 -->
        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="50%"
            @close='setRightDialogClosed'
        >
            <el-tree
                :data="rightsList"
                :props="treeProps"
                show-checkbox
                node-key="id"
                ref="treeRef"
                :default-checked-keys="defKeys"
            >
            </el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false"
                    >取 消</el-button
                >
                <el-button type="primary" @click="allotRights"
                    >确 定</el-button
                >
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: 'Roles',
    data() {
        return {
            roleList: [],
            updateRoleId: 0,
            dtype: 'add',
            // 添加角色视图显示
            addRolesVisible: false,
            // 分配权限视图显示
            setRightDialogVisible: false,
            // 添加角色表单数据
            addRolesForm: {
                roleName: '',
                roleDesc: ''
            },
            // 添加角色表单规则
            addRolesFormRules: {
                roleName: [
                    {
                        required: true,
                        message: '请输入角色名称',
                        trigger: 'blur'
                    }
                ]
            },
            // 权限列表
            rightsList: [],
            //树形控件
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            // 默认选择节点id值数组
            defKeys:[],
            roleId: ''
        };
    },
    mounted() {
        this.getRolesList();
    },
    methods: {
        // 获取角色列表
        async getRolesList() {
            const { data: res } = await this.$http.get('roles');
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败');
            }
            this.roleList = res.data;
        },
        // 通过递归的形式，获取角色下所有三级权限的id，并保持到defkeys数组内。
        getLeafKeys(node, arr){
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => this.getLeafKeys(item, arr))
        },
        // 监听分配权限对话框关闭
        setRightDialogClosed(){
            this.defKeys = []
        },
        // 分配权限按钮事件
        async showSetRightDialog(role) {
            // 获取所有权限列表
            const { data: res } = await this.$http.get('rights/tree');
            if (res.meta.status !== 200) {
                return this.$message.error('获取权限列表失败');
            }
            this.rightsList = res.data;
            this.roleId = role.id;
            this.getLeafKeys(role, this.defKeys)
            this.setRightDialogVisible = true;
        },
        // 关闭添加角色视图时清空表单
        addRoleClose() {
            this.$refs.addRolesFormRef.resetFields();
            this.addRolesForm.roleName = '';
            this.addRolesForm.roleDesc = '';
        },
        // 视图内确认添加/编辑角色按钮事件
        addOrUpdateRole() {
            if (this.dtype == 'add') {
                this.$refs.addRolesFormRef.validate(async (valid) => {
                    if (!valid) {
                        return;
                    }
                    const { data: res } = await this.$http.post(
                        'roles',
                        this.addRolesForm
                    );
                    if (res.meta.status !== 201) {
                        return this.$message.error('添加角色失败');
                    }
                    this.$message.success('添加角色成功');
                    this.addRolesVisible = false;
                    this.getRolesList();
                });
            } else {
                this.$refs.addRolesFormRef.validate(async (valid) => {
                    // 验证不通过返回
                    if (!valid) {
                        return;
                    }
                    console.log(this.addRolesForm.roleDesc === '');
                    const { data: res } = await this.$http.put(
                        'roles/' + parseInt(this.updateRoleId),
                        {
                            roleName: this.addRolesForm.roleName,
                            roleDesc: this.addRolesForm.roleDesc
                        }
                    );
                    if (res.meta.status !== 200) {
                        return this.$message.error('修改角色信息失败');
                    }
                    this.$message.success('修改角色信息成功');
                    this.addRolesVisible = false;
                    this.getRolesList();
                });
            }
        },
        // 编辑角色按钮
        addRole() {
            console.log(this);
            this.addRolesVisible = true;
            this.dtype = 'add';
        },
        // 删除角色按钮事件
        removeRoleById(id) {
            this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
            })
                .then(async () => {
                    const { data: res } = await this.$http.delete(
                        'roles/' + id
                    );
                    if (res.meta.status !== 200) {
                        return this.$message({
                            type: 'error',
                            message: '删除失败!'
                        });
                    }
                    this.getRolesList();
                    return this.$message({
                        type: 'success',
                        message: '删除成功!'
                    });
                })
                .catch(() => {
                    return this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
        },
        // 编辑角色按钮事件
        updateRoleById(role) {
            this.addRolesVisible = true;
            this.dtype = 'update';
            this.addRolesForm.roleName = role.roleName;
            this.addRolesForm.roleDesc = role.roleDesc;
            this.updateRoleId = role.id;
        },
        // 根据id删除对应权限
        removeRightById(row, rightId) {
            console.log(row, rightId);
            this.$confirm('此操作将取消该角色的该项权限, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning',
                center: true
            })
                .then(async () => {
                    const { data: res } = await this.$http.delete(
                        `roles/${row.id}/rights/${rightId}`
                    );
                    if (res.meta.status !== 200) {
                        return this.$message({
                            type: 'error',
                            message: '删除失败!'
                        });
                    }
                    row.children = res.data;
                    return this.$message({
                        type: 'success',
                        message: '删除成功!'
                    });
                })
                .catch(() => {
                    return this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
        },
        async allotRights(){
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys(),
            ]
            const idStr = keys.join(',');
            const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
            if (res.meta.status !== 200) {
                return this.$message.error('分配权限失败');
            }
            this.$message.success('分配权限成功');
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
};
</script>

<style scoped lang='less'>
.el-tag {
    margin: 7px;
}
.bdbottom:first-child {
    border-top: 1px solid #eee;
}
.bdbottom {
    border-bottom: 1px solid #eee;
}
.bdtbottom:not(:first-child) {
    border-top: 1px solid #eee;
}
.vcenter {
    display: flex;
    align-items: center;
}
</style>