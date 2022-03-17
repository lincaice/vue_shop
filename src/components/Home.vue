<template>
    <el-container class="home-container">
        <el-header>
            <div>
                <span>
                    <img src="../assets/电商.png" alt="" />
                </span>
                <span>电商后台管理系统</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <el-container>
            <!-- 页面主体 -->
            <el-aside :width="isCollapse ? '64px':'200px'">
                <div class="toggleButton" @click="toggleCollapse">
                    <i class="iconfont icon-zhedie"></i>
                </div>
                <!-- 侧边栏菜单区 -->
                <el-menu
                    background-color="#313743"
                    text-color="#fff"
                    active-text-color="skyblue"
                    :unique-opened="true"
                    :collapse="isCollapse"
                    :collapse-transition='false'
                    :default-active="activePath"
                    router
                >
                    <!-- 一级菜单 -->
                    <el-submenu
                        :index="'' + item.id"
                        v-for="item in menuList"
                        :key="item.id"
                    >
                        <!-- 一级菜单模板区 -->
                        <template slot="title">
                            <!-- 图标 -->
                            <i :class="iconObj[item.id]"></i>
                            <span>{{ item.authName }}</span>
                        </template>
                        <!-- 二级菜单模板区 -->
                        <el-menu-item
                            :index="'/' + subItem.path"
                            v-for="subItem in item.children"
                            :key="subItem.id"
                            @click="saveNavState('/' + subItem.path)"
                        >
                            <template slot="title">
                                <!-- 图标 -->
                                <i class="el-icon-menu"></i>
                                <span>{{ subItem.authName }}</span>
                            </template>
                        </el-menu-item>
                    </el-submenu>
                </el-menu>
            </el-aside>
            <!-- 右侧内容主体 -->
            <el-main>
                <router-view></router-view>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
export default {
    name: 'Home',
    data() {
        return {
            menuList: [],
            iconObj: {
                125: 'iconfont icon-user',
                103: 'iconfont icon-tijikongjian',
                101: 'iconfont icon-shangpin',
                102: 'iconfont icon-danju',
                145: 'iconfont icon-baobiao'
            },
            isCollapse: false,
            // 被激活的链接地址
            activePath: ''
        };
    },
    methods: {
        logout() {
            window.sessionStorage.clear();
            this.$router.push('/login');
        },
        // 获取所有菜单
        async getMenuList() {
            const { data: res } = await this.$http.get('menus');
            if (res.meta.status !== 200) {
                return this.$message.error(res.meta.msg);
            }
            this.menuList = res.data;
        },
        // 点击按钮，切换菜单折叠与展开
        toggleCollapse(){
            this.isCollapse = !this.isCollapse;
        },
        // 保存链接的激活状态
        saveNavState(activePath){
            window.sessionStorage.setItem('activePath', activePath);
            this.activePath = activePath;
        }
    },
    mounted() {
        this.getMenuList();
        this.activePath = window.sessionStorage.getItem('activePath');
    }
};
</script>

<style scoped lang='less'>
.home-container {
    height: 100%;
}
// 头部菜单样式
.el-header {
    background-color: #363d40;
    display: flex;
    justify-content: space-between;
    align-items: center;
    > div {
        // display: flex;
        // align-items: center;
        height: 60px;
        line-height: 60px;
        img {
            width: 40px;
            height: 40px;
            margin-right: 20px;
            vertical-align: middle;
        }
        span {
            color: white;
            font-size: 20px;
        }
    }
}
.el-aside {
    background-color: #313743;
    transition: .2s width;
    .el-menu {
        border-right: none;
    }
    .toggleButton{
        height: 30px;
        line-height: 30px;
        text-align: center;
        background-color: #495068;
        cursor: pointer;
    }
}
.el-main {
    background-color: #eee;
}
</style>
