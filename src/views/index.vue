<template>
<div class="main">
    <div class="interaction rr-block">
        <el-button v-if="!showReplay" type="primary" @click="record">录制</el-button>
        <el-button v-if="!showReplay" type="success" @click="replay">回放</el-button>
        <el-button v-if="showReplay" type="warning" @click="reset">返回演示</el-button>
    </div>
    <div v-if="!showReplay">
        <div class="form">
            <el-form ref="form" :rules="formRules" :model="form" label-width="80px">
                <el-form-item label="活动名称" prop="name">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="活动区域" prop="region">
                    <el-select v-model="form.region" placeholder="请选择活动区域">
                        <el-option label="区域一" value="shanghai"></el-option>
                        <el-option label="区域二" value="beijing"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="活动时间">
                    <el-col :span="11" prop="date1">
                        <el-date-picker type="date" placeholder="选择日期" v-model="form.date1" style="width: 100%;"></el-date-picker>
                    </el-col>
                    <el-col class="line" :span="2">-</el-col>
                    <el-col :span="11" prop="date2">
                        <el-time-picker placeholder="选择时间" v-model="form.date2" style="width: 100%;"></el-time-picker>
                    </el-col>
                </el-form-item>
                <el-form-item label="即时配送">
                    <el-switch v-model="form.delivery"></el-switch>
                </el-form-item>
                <el-form-item label="活动性质" prop="type">
                    <el-checkbox-group v-model="form.type">
                        <el-checkbox label="美食/餐厅线上活动" name="type"></el-checkbox>
                        <el-checkbox label="地推活动" name="type"></el-checkbox>
                        <el-checkbox label="线下主题活动" name="type"></el-checkbox>
                        <el-checkbox label="单纯品牌曝光" name="type"></el-checkbox>
                    </el-checkbox-group>
                </el-form-item>
                <el-form-item label="特殊资源" prop="resource">
                    <el-radio-group v-model="form.resource">
                        <el-radio label="线上品牌商赞助"></el-radio>
                        <el-radio label="线下场地免费"></el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="活动形式" prop="desc">
                    <el-input type="textarea" v-model="form.desc"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="onSubmit">打印表单数据</el-button>
                    <el-button type="danger" @click="logError">手动抛错</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
    <div ref="replayer"></div>
</div>
</template>

<script>
const rrweb = require("rrweb")
import rrwebPlayer from 'rrweb-player'
export default {
    data() {
        return {
            eventsMatrix : [[]],  // 使用二维数组来存放多个 event 数组
            showReplay: false,
            form: {
                name: '',
                region: '',
                date1: '',
                date2: '',
                delivery: false,
                type: [],
                resource: '',
                desc: '',
            },
            formRules: {
                name: [{
                    required: true,
                    message: '请输入活动名称',
                    trigger: 'blur'
                }, ],
                region: [{
                    required: true,
                    message: '请选择活动区域',
                    trigger: 'change'
                }],
                date1: [{
                    required: true,
                    message: '请选择日期',
                    trigger: 'change'
                }],
                date2: [{
                    required: true,
                    message: '请选择时间',
                    trigger: 'change'
                }],
                type: [{
                    required: true,
                    message: '请选择活动性质',
                    trigger: 'change'
                }],
                resource: [{
                    required: true,
                    message: '请选择活动资源',
                    trigger: 'change'
                }],
                desc: [{
                    required: true,
                    message: '请填写活动形式',
                    trigger: 'blur'
                }],
            },
            stopFn: null,
        }
    },
    methods: {
        onSubmit() {
            console.log(JSON.stringify(this.form))
        },
        logError() {
            throw Error('模拟错误')
        },
        record() {
            const self = this
            this.stopFn = rrweb.record({
                checkoutEveryNth: 100, // 每 100 个 event 重新制作快照
                emit(event, isCheckout) {
                    // isCheckout 是一个标识，告诉你重新制作了快照
                    if (isCheckout) {
                        self.eventsMatrix.push([]);
                    }
                    const lastEvents = self.eventsMatrix[self.eventsMatrix.length - 1];
                    lastEvents.push(event);
                },
                // 定制的选项
                plugins: [rrweb.getRecordConsolePlugin({
                        level: ["info", "log", "warn", "error"],
                        lengthThreshold: 10000,
                        stringifyOptions: {
                        stringLengthLimit: 1000,
                        numOfKeysLimit: 100,
                        depthOfLimit: 1
                    },
                    logger: window.console,
                })],
            });
        },
        replay() {
            console.log('最近的操作记录: ', JSON.stringify(this.eventsMatrix[this.eventsMatrix.length - 1]));
            if(this.eventsMatrix[this.eventsMatrix.length - 1].length<=0) return this.$message.error("请先点击录制按钮进行录制！");
            this.stopFn();
            this.showReplay = true
            new rrwebPlayer({
                target: this.$refs.replayer, // 可以自定义 DOM 元素
                // 配置项
                props: {
                    logConfig: true,
                    events: this.eventsMatrix[this.eventsMatrix.length - 1],
                    plugins: [
                        rrweb.getReplayConsolePlugin({
                        level: ['info', 'log', 'warn', 'error'],
                    }),
                ],
                },
            });
        },
        reset() {
            this.showReplay = false
            location.reload()
        }
    }
}
</script>

<style lang="scss" scoped>
.main {
    padding: 20px;

    .interaction {
        padding: 10px;
    }

    .form {
        width: 600px;
        margin: 20px auto;

        ::v-deep .el-form-item__content {
            text-align: left;
        }
    }
}
</style>
