<template>
    <div class="login_container">
        <div class="login_box">
            <!-- 头像区域 -->
            <div class="avatar_box">
                <img src="../assets/logo.png" />
            </div>
            <!-- 登录表单区 -->
            <el-form
                class="login_form"
                label-width="0px"
                :model="loginForm"
                ref="loginFormRef"
                :rules="loginFormRules"
            >
                <!-- 用户名 -->
                <el-form-item prop="username">
                    <el-input
                        v-model="loginForm.username"
                        prefix-icon="el-icon-user"
                    ></el-input>
                </el-form-item>
                <!-- 密码 -->
                <el-form-item prop="password">
                    <el-input
                        v-model="loginForm.password"
                        type="password"
                        prefix-icon="el-icon-lock"
                    ></el-input>
                </el-form-item>
                <!-- 按钮区域 -->
                <el-form-item class="btns">
                    <el-button type="primary" @click="login">登录</el-button>
                    <el-button type="info" @click="resetLoginFrom"
                        >重置</el-button
                    >
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            //表单信息
            loginForm: {
                username: "admin",
                password: "123456",
            },
            // 表单验证规则
            loginFormRules: {
                // 验证用户名
                username: [
                    {
                        required: true,
                        message: "请输入登录名称",
                        trigger: "blur",
                    },
                    {
                        min: 3,
                        max: 10,
                        message: "长度在 3 到 10 个字符",
                        trigger: "blur",
                    },
                ],
                // 验证密码
                password: [
                    {
                        required: true,
                        message: "请输入登录密码",
                        trigger: "blur",
                    },
                    {
                        min: 6,
                        max: 15,
                        message: "长度在 6 到 15 个字符",
                        trigger: "blur",
                    },
                ],
            },
        };
    },
    methods: {
        resetLoginFrom() {
            this.$refs.loginFormRef.resetFields();
            console.log();
        },
        login() {
            this.$refs.loginFormRef.validate(async (valid) => {
                if (valid) {
                    const { data: res } = await this.$http.post(
                        "login",
                        this.loginForm
                    );
                    if (res.meta.status != 200) {
                        return this.$message.error("登录失败");
                    }
                    this.$message.success("登录成功");
                    window.sessionStorage.setItem('token',res.data.token);
                    this.$router.push('/home');
                }
            });
        },
    },
};
</script>

<style lang='less' scoped>
.login_container {
    background-color: #65b2c9;
    height: 100%;
}

.login_box {
    background-color: #fff;
    width: 500px;
    height: 350px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    border-radius: 5px;
    .avatar_box {
        height: 130px;
        width: 130px;
        border: 1px solid #eee;
        border-radius: 50%;
        box-shadow: 0 0 10px #ddd;
        padding: 10px;
        margin: -65px auto 0 auto;
        background-color: #fff;
        img {
            border-radius: 50%;
            width: 100%;
            height: 100%;
            background-color: #eee;
        }
    }
}

.login_form {
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 15px;
    box-sizing: border-box;
}
.btns {
    display: flex;
    justify-content: space-around;
}
</style>