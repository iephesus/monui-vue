<template>
    <div>
        <h3 class="mar10" v-if="machineInfo.Machine">机器信息:<span class="mar10">{{machineInfo.Machine}}</span><span
                class="mar10">{{machineInfo.Ip}}</span></h3>
        <div id="app">
            <Login v-if="!isLogin"></Login>
            <ShowTable v-if="configs.length !== 0" :configs="configs"></ShowTable>
        </div>
    </div>
</template>

<script>
    import ShowTable from "./components/ShowTable"
    import Login from "./components/Login";

    export default {
        name: 'App',
        components: {
            Login,
            ShowTable
        },
        data: () => {
            return {
                configs: [],
                machineInfo: {},
                base: process.env.VUE_APP_BASE,
                isLogin: false
            }
        },
        created() {
            if (sessionStorage.getItem("isLogin") === "true") {
                this.isLogin = true
            } else {
                this.isLogin = false
            }
            this.isLoginSuccess()
        },
        methods: {
            isLoginSuccess() {
                if (sessionStorage.getItem("isLogin") === "true") {
                    this.getAllSv()
                    this.getMachineInfo()
                }
            },
            getAllSv() {
                const base = this.base
                fetch(`${base}/supervisors`, {
                    method: 'get',
                }).then((r) => r.json()).then((r) => Promise.all(r.map((name) => {
                        return fetch(`${base}/supervisor/` + encodeURIComponent(name), {
                            method: 'get',
                        }).then((data) => {
                            return data.json();
                        })
                    }))
                ).then((configs) => {
                    this.configs = configs
                }).catch(function (err) {
                    console.error(err);
                })
            },
            getMachineInfo() {
                const base = this.base
                fetch(`${base}/info`, {
                    method: 'get',
                }).then(r => r.json()).then(r => {
                    this.machineInfo = r
                }).catch(err => console.error(err))
            }
        }
    }
</script>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }

    .mar10 {
        margin-top: 10px;
        margin-left: 15px;
    }
</style>
