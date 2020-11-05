<template>
    <div id="app">
        <ShowTable v-if="configs.length !== 0" :configs="configs"></ShowTable>
    </div>
</template>

<script>
    import ShowTable from "./components/ShowTable"

    export default {
        name: 'App',
        components: {
            ShowTable
        },
        data: () => {
            return {
                configs: [],
                base: process.env.VUE_APP_BASE,
            }
        },
        created() {
            this.getAllSv()
        },
        methods : {
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
</style>
