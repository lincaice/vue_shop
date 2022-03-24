<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加搜索用户区 -->
      <el-row :gutter="20">
        <el-col :span="6">
          <el-input placeholder="请输入内容" clearable v-model="queryInfo.query">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>x
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goGoodsAdd">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table表格区 -->
      <el-table :data="goodsList" style="width: 100%" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称" width="width"></el-table-column>
        <el-table-column prop="goods_price" label="商品价格（元）" width="90px"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="70px"></el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="200px">
          <template v-slot="scope">{{scope.row.add_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template v-slot="scope">
            <el-button type="primary" icon="el-icon-edit" size="small"></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="small"
              @click="removeById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'List',
  data() {
    return {
      // 列表展示信息
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表信息
      goodsList: [],
      // 列表总数
      total: 0
    };
  },
  methods: {
    // 获取商品列表信息
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      });
      if (res.meta.status !== 200) {
        return this.$message({
          message: '获取商品列表失败',
          type: 'error'
        });
      }
      this.goodsList = res.data.goods;
      this.total = res.data.total;
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getGoodsList();
    },
    // 删除商品
    removeById(goodId) {
      this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(`goods/${goodId}`);
          if (res.meta.status !== 200) {
            return this.$message.error('删除商品失败');
          }
          this.$message({
            type: 'success',
            message: '删除商品成功!'
          });
          this.getGoodsList();
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
    },
    goGoodsAdd() {
      this.$router.push('/goods/add');
    }
  },
  mounted() {
    this.getGoodsList();
  }
};
</script>

<style>
</style>