<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-row :gutter="10">
        <el-col :span="6">
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格区 -->
      <tree-table
        style="margin-top: 15px"
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        border
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            style="color: lightgreen"
            v-if="scope.row.cat_deleted === false"
          ></i>
          <i class="el-icon-error" style="color: red" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" v-else-if="scope.row.cat_level === 2">三级</el-tag>
        </template>
        <template slot="opt" slot-scope>
          <el-button type="primary" size="small" icon="el-icon-edit">编辑</el-button>
          <el-button type="danger" size="small" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
      <!-- 添加角色视图 -->
      <el-dialog
        title="添加分类"
        :visible.sync="addCateDialogVisible"
        width="30%"
        @close="addCateClose"
      >
        <!-- 主体区域 -->
        <el-form
          ref="addCateFormRef"
          :model="addCateForm"
          :rules="addCateFormRules"
          label-width="90px"
          status-icon
          status
        >
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类">
            <el-cascader
              clearable
              change-on-select
              v-model="selectedKeys"
              :options="ParentCateList"
              :props="{
                                expandTrigger: 'hover',
                                ...cascaderProps
                            }"
              @change="parentCateChange"
            ></el-cascader>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'Cate',
  data() {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //商品分类列表
      cateList: [],
      // 总条数
      total: 0,
      // 为table指定类定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      // 监听添加分类对话框显示与隐藏属性
      addCateDialogVisible: false,
      // 添加分类表单
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        // 默认一级分类
        cat_level: 0
      },
      // 分类规则数组
      addCateFormRules: {
        cat_name: [
          {
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }
        ]
      },
      //父级数据列表
      ParentCateList: [],
      // 指定级联选择器配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中父级分类id数组
      selectedKeys: []
    };
  },
  methods: {
    // 获取商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      });
      if (res.meta.status !== 200) {
        this.$message.error('获取商品分类失败');
      }
      this.cateList = res.data.result;
      this.total = res.data.total;
    },
    // 监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    // 监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    // 添加分类对话框的确定按钮事件
    addCate() {
      this.$refs.addCateFormRef.validate(async (valid) => {
        if (!valid) {
          return;
        }
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        );
        if (res.meta.status !== 201) {
          this.$message.error('添加分类失败');
        }
        this.$message.success('添加分类成功！');
        this.getCateList();
        this.addCateDialogVisible = false;
      });
    },
    // 添加分类对话框的关闭事件
    addCateClose() {
      this.$refs.addCateFormRef.resetFields();
      this.selectedKeys = [];
      this.addCateForm.cat_level = 0;
      this.addCateForm.cat_pid = 0;
    },
    // 显示添加分类对话框事件
    showAddCateDialog() {
      this.getParentCateList();
      this.addCateDialogVisible = true;
    },
    // 获取父级节点事件
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      });
      if (res.meta.status !== 200) {
        this.$message.error('获取父级列表失败');
      }
      this.ParentCateList = res.data;
    },
    // 选择框触发事件
    parentCateChange() {
      console.log(this.selectedKeys);
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid =
          this.selectedKeys[this.selectedKeys.length - 1];
        this.addCateForm.cat_level = this.selectedKeys.length;
        return;
      } else {
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      }
    }
  },
  mounted() {
    this.getCateList();
  }
};
</script>

<style scoped>
.el-cascader {
  width: 100%;
}
</style>