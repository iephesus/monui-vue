<template>
    <div>
        <Card class="card">
            <div style="text-align:center">
                <img class="loginImage" src="../assets/loginLogo.png">
                <h3>请登录</h3>
            </div>
        </Card>
        <Form ref="LoginForm" :model="LoginData" :rules="ruleInline" inline>
            <FormItem prop="name">
                <Input type="text" v-model="LoginData.name" placeholder="Username">
                    <Icon type="ios-person-outline" slot="prepend"></Icon>
                </Input>
            </FormItem>
            <FormItem prop="password">
                <Input type="password" v-model="LoginData.password" placeholder="Password">
                    <Icon type="ios-lock-outline" slot="prepend"></Icon>
                </Input>
            </FormItem>
            <FormItem>
                <Button type="primary" @click="handleSubmit('LoginForm')">登录</Button>
            </FormItem>
        </Form>
    </div>
</template>

<script>
    export default {
        name: "Login",
        data: () => {
            return {
                base: process.env.VUE_APP_BASE,
                isLogin: false,
                LoginData: {
                    name: '',
                    password: ''
                },
                ruleInline: {
                    name: [
                        {required: true, message: '请输入用户名', trigger: 'blur'}
                    ],
                    password: [
                        {required: true, message: '请输入密码', trigger: 'blur'},
                        // { type: 'string', min: 6, message: 'The password length cannot be less than 6 bits', trigger: 'blur' }
                    ]
                }
            }
        },
        methods: {
            handleSubmit(name) {
                let data = new FormData;
                data.append("name", this.LoginData.name)
                data.append("password", this.LoginData.password)
                const base = this.base
                this.$refs[name].validate((valid) => {
                    if (valid) {
                        fetch(`${base}/login`, {
                                method: 'post',
                                body: data
                            }
                        ).then(r => {
                            if (r.status !== 200) {
                                this.$Message.error("登录错误！");
                            } else {
                                r.json().then(r => {
                                    this.isLogin = r.loginStatus
                                    sessionStorage.setItem("isLogin", this.isLogin)
                                    this.$parent.isLogin = this.isLogin
                                    if (this.isLogin) {
                                        this.$parent.isLoginSuccess()
                                        this.$Message.success(r.info);
                                    } else {
                                        this.$Message.error(r.info);
                                    }
                                })
                            }
                        })
                    }
                })
            }
        }
    }
</script>

<style scoped>
    .card {
        margin: 200px auto 50px auto;
        width: 350px;
    }

    .loginImage {
        width: 100px;
        height: 100px;
    }
</style>