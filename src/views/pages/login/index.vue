//*登录组件
<template>
    <div class="login-container">
        <!-- model为双向绑定的数据 rules为表单的验证规则 -->
        <el-form ref="form" :model="form" :rules="rule" class="login-box">
            <h2 class="box-title">系统登录</h2>
            <el-form-item class="box-item" prop="userName">
                <el-input
                    placeholder="账户"
                    prefix-icon="el-icon-user"
                    v-model="form.userName">
                </el-input>
            </el-form-item>
            <el-form-item class="box-item" prop="userPassword">
                <el-input
                    :type="isPassword ? 'password' : ''"
                    placeholder="密码"
                    v-model="form.userPassword">
                    <i slot="prefix" class="el-input__icon el-icon-lock"></i>
                    <el-button slot="suffix" type="text" @click="onChangePassword">
                        <svg class="icon el-input__icon" aria-hidden="true" v-show="isPassword">
                            <use xlink:href="#icon-yincangmima"></use>
                        </svg>
                        <svg class="icon el-input__icon" aria-hidden="true" v-show="!isPassword">
                            <use xlink:href="#icon-xianshimima"></use>
                        </svg>
                    </el-button>
                </el-input>
            </el-form-item>
            <el-form-item class="box-item" prop="userVerifyCode">
                <el-input
                    placeholder="验证码"
                    v-model="form.userVerifyCode">
                    <svg class="icon el-input__icon" slot="prefix" aria-hidden="true">
                        <use xlink:href="#icon-yanzhengma1"></use>
                    </svg>
                    <img-verify slot="suffix" :changeCode.sync='verifyCode'></img-verify>
                </el-input>
            </el-form-item>
            <el-form-item class="box-item">
                <el-button :loading="loadingLogin" class="item-btn" type="primary" @click="onClickSubmit">登录</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>

<script>
import { mapMutations } from 'vuex'
import INTERFACE from '@/api/interface.js'
export default {
    data() {
        const checkCode = (rule, value, callback) => {
            if(!value) {
                return callback(new Error('请输入'))
            } else {
                if (this.verifyCode.toLowerCase() !== this.form.userVerifyCode.toLowerCase()) {
                    return callback(new Error('请输入正确的验证码！'))
                } else {
                    callback()
                }
            }
        }
        return {
            verifyCode: '',
            form: {
                userName: '',
                userPassword: '',
                userVerifyCode: '',
            },
            rule: {
                userName: [ { required: true, message: '请输入', trigger: 'blur' }],
                userPassword: [ { required: true, message: '请输入', trigger: 'blur' }],
                userVerifyCode: [ { validator: checkCode, trigger: 'blur' }],
            },
            isPassword: true,
            loadingLogin: false,
        };
    },
    methods: {
        ...mapMutations(['setUser']),

        onChangePassword() {
            this.isPassword = ! this.isPassword
        },
        // 加上表单校验 两个要素：通过rules属性传入验证规则，并添加form-item每项的prop属性名。
        onClickSubmit() {
            this.$refs.form.validate(v => v && this.handleLogin())
        },
        async handleLogin() {
            this.loadingLogin = true
            let res = await INTERFACE.login(this.form)
            this.loadingLogin = false
            if (!res.data) {
                this.$message.error('用户名或密码错误！')
                return false
            } else {
                this.setUser(res.data)
                sessionStorage.setItem('login','yes');
                this.$router.push({ path: '/welcome' })
            }
        },
    },
};
</script>

<style lang='scss' scoped>
.login-container {
    width: 100%;
    height: 100%;
    position: relative;
    overflow: hidden;
    background: url("@/assets/images/login/4.jpg") no-repeat center / cover;
}
.login-box {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    width: 400px;
    height: auto;
    padding: 0 40px;
    box-sizing: border-box;
    border-radius: 4px;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    background-color: rgba($color: #fff, $alpha: 0.84);
    .box-title {
        margin: 10px auto;
    }
    .box-item {
        width: 100%;
        color: #fff;
        text-align: center;
        .item-btn {
            width: 50%;
            border-radius: 20px;
        }
    }
}
</style>
