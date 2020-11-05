<template>
    <div>
        <Table stripe highlight-row :columns="monColumns" :data="svdata">
            <template slot-scope="{row, index}" slot="status">
                <Tag v-if="svStatusArray[index].running === null" color="warning" size="large">{{svStatusArray[index].status}}</Tag>
                <Tag v-else-if="svStatusArray[index].running" color="success" size="large">{{svStatusArray[index].status}}</Tag>
                <Tag v-else color="error" size="large">{{svStatusArray[index].status}}</Tag>
            </template>
            <template slot-scope="{row, index}" slot="description">
                <List size="small">
                    <ListItem v-if="configs[index].WorkDir">&nbsp;at: {{configs[index].WorkDir}}</ListItem>
                    <ListItem v-for="(value,name,index) of configs[index].Environment"
                              v-bind:key="index">
                        <span style="font-size: 0.5em; margin-right: 5px">{{name}}:&nbsp;</span><span
                            style="font-size: 0.5em">{{value}}</span>
                    </ListItem>
                </List>
            </template>
            <template slot-scope="{row, index}" slot="info">
                <List size="small">
                    <ListItem><span style="color: #19be6b; margin-right: 5px">Restart:</span>{{configs[index].Restart === -1 ? '∞' : config.Restart }}
                    </ListItem>
                    <ListItem><span style="color: #2d8cf0; margin-right: 5px">Restart Timeout:</span>{{(configs[index].RestartTimeout/1000000000).toFixed(2)}}s
                    </ListItem>
                    <ListItem><span style="color: #ff9900; margin-right: 5px">Stop Timeout:</span>{{(configs[index].StopTimeout/1000000000).toFixed(2)}}s
                    </ListItem>
                </List>
            </template>
            <template slot-scope="{ row, index }" slot="action">
                <Poptip confirm transfer title="确定吗?" @on-ok="start(index)">
                    <Button :disabled="svStatusArray[index].running !== false" :loading="svStatusArray[index].loading" type="primary" size="small" style="margin-right: 5px">Start
                    </Button>
                </Poptip>
                <Poptip confirm transfer title="确定吗?" @on-ok="stop(index)">
                    <Button :disabled="svStatusArray[index].running !== true" :loading="svStatusArray[index].loading" type="error" size="small" style="margin-right: 5px">Stop
                    </Button>
                </Poptip>
                <Button type="info" size="small" @click="showlogs(index)">Log
                </Button>
            </template>
        </Table>
        <Modal scrollable draggable ok-text="下载日志"
               v-model="isShowModal"
               title="Log Details"
               @on-ok="downloadLog(modalSvName)">
            <p>{{modalSvName}}</p>
            <pre style="height: 300px; overflow-y: scroll" v-text="logs"></pre>
        </Modal>
    </div>
</template>

<script>
    export default {
        name: "ShowTable",
        props: {
            configs: Array
        },
        created() {
            this.initData()
            this.interval = setInterval(() => {
                this.updateData()
            }, 2000)
        },
        destroyed() {
            clearInterval(this.interval)
        },
        data: () => {
            return {
                isShowModal: false,
                modalSvName: '',
                logs: ``,
                svStatusArray: [], //状态数组
                base: process.env.VUE_APP_BASE,
                monColumns: [
                    {
                        type: 'index',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: '服务名',
                        width: 200,
                        key: 'name',
                        align: 'center'
                    },
                    {
                        title: '状态',
                        slot: 'status',
                        align: 'center',
                        width: 150
                    },
                    {
                        title: '命令参数',
                        key: 'command',
                        align: 'center'
                    },
                    {
                        title: '路径及变量',
                        slot: 'description',
                        align: 'center'
                    },
                    {
                        title: '信息',
                        slot: 'info',
                        align: 'center',
                        width: 230
                    },
                    {
                        title: '操作',
                        slot: 'action',
                        align: 'center',
                        width: 250
                    }
                ],
                svdata: []
            }
        },
        methods: {
            initData() {
                this.configs.forEach(function (v) {
                    this.svStatusArray.push({
                        name: v.Name,
                        status: function (v) {
                            return v.running ? "Start" : "Stop"
                        }(v),
                        running: v.running,
                        loading: true
                    })
                    this.svdata.push({
                        name: v.Name,
                        status: function (v) {
                            return v.running ? "Start" : "Stop"
                        }(v),
                        command: function (v) {
                            let com = v.Command
                            let args = v.Args !== null ? v.Args.join(' ') : ''
                            return com + "  " + args
                        }(v),
                        description: function (v) {
                            let workdir = v.WorkDir !== null ? v.WorkDir : ''
                            return "at: " + workdir
                        }(v),
                    })
                }, this)
            },
            updateData() {
                this.svStatusArray.forEach(function (v, i) {
                    this.fetchState(v, i)
                }, this)
            },
            start(index) {
                let st = this.svStatusArray[index]
                let base = this.base
                if (st.running) return;
                st.running = null
                st.status = "Pending"
                st.loading = true
                return fetch(`${base}/supervisor/` + encodeURIComponent(st.name), {
                    method: 'post',
                }).then((r) => {
                    if (r.status !== 200) {
                        console.warn(r.status, r.statusText)
                    } else {
                        return this.fetchState(st,index);
                    }
                }).catch((err) => {
                    console.error(st.name, err);
                })
            },
            stop(index) {
                let st = this.svStatusArray[index]
                let base = this.base
                if (!st.running) return;
                st.running = null
                st.status = "Pending"
                st.loading = true
                return fetch(`${base}/instance/` + encodeURIComponent(st.name), {
                    method: 'post',
                }).then((r) => {
                    if (r.status !== 204) {
                        console.warn(r.status, r.statusText)
                    } else {
                        return this.fetchState(st,index);
                    }
                }).catch((err) => {
                    console.error(st.name, err);
                })
            },
            showlogs(index) {
                let base = this.base
                let name = this.configs[index].Name
                fetch(`${base}/supervisor/${name}/log`, {
                    method: 'get',
                }).then(res => {
                    if (res.status === 404) {
                        this.isShowModal = false
                        this.logs = ``
                        this.$Message["error"]({
                            background: true,
                            content: '配置未设置logFile，找不到文件...',
                            duration: 3
                        })
                    } else {
                        this.isShowModal = true
                        this.modalSvName = this.configs[index].Name
                        res.text().then(r => {
                            this.logs = r
                        })
                    }
                })
            },
            downloadLog(name) {
                let base = this.base
                fetch(`${base}/supervisor/${name}/log`, {
                    method: 'get',
                }).then(res => {
                    if (res.status === 404) {
                        this.$Message["error"]({
                            background: true,
                            content: '找不到文件...',
                            duration: 3
                        })
                        return
                    }
                    const filename = name + ".log"

                    res.blob().then(b => {
                        const link = document.createElement('a')
                        link.style.display = 'none'
                        link.download = filename
                        link.target = '_blank'
                        link.href = URL.createObjectURL(b)
                        document.body.appendChild(link)
                        link.click()
                        URL.revokeObjectURL(link.href)
                        document.body.removeChild(link)
                    })
                })
            },
            fetchState(config, index) {
                const name = config.name
                const base = this.base
                return fetch(`${base}/instance/` + encodeURIComponent(name), {
                    method: 'get',
                }).then((r) => {
                    if (r.status === 404) {
                        return {"running": false}
                    }
                    return r.json()
                }).then((reply) => {
                    this.svStatusArray[index].running = reply.running
                    if(reply.running) {
                        this.svStatusArray[index].status = "Start"
                    } else {
                        this.svStatusArray[index].status = "Stop"
                    }
                    this.svStatusArray[index].loading = false
                }).catch((err) => {
                    console.error(name, err);
                })
            }
        },

    }
</script>

<style scoped>

</style>