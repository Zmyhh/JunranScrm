<template>
    <div class="audit">

        <template>
            <div style="padding: 1%;text-align: left">
                <el-input style="width: 20%" v-model="actiName" placeholder="根据关键字搜索"></el-input>
                <el-date-picker
                        v-model="auditStartTime"
                        type="datetime"
                        placeholder="起始时间"
                        value-format="yyyy-MM-dd HH:mm:ss"
                >
                </el-date-picker>
                <el-date-picker
                        v-model="auditEndTime"
                        type="datetime"
                        value-format="yyyy-MM-dd HH:mm:ss"
                        placeholder="结束时间">
                </el-date-picker>


                <el-select v-model="value" placeholder="根据状态查询">
                    <el-option
                            v-for="item in options"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value">
                    </el-option>
                </el-select>




                <el-button @click="search">查询</el-button>
                <el-button @click="BatchThrough">批量通过</el-button>
            </div>
            <el-table
                    :data="acditList"
                    style="width: 100%"
                    @selection-change="selChange"
                    border
            >
                <el-table-column type="expand">
                    <template slot-scope="props">
                        <el-form label-position="left" inline class="demo-table-expand">

                            <el-form-item label="好评图片">
                                <img style="width: 20%" :src="'http://'+props.row.img" alt="用户好评图片">
                            </el-form-item>
                            <br/>
                            <el-form-item label="活动名称">
                                {{props.row.activityName}}
                            </el-form-item>

                            <el-form-item label="审核时间">

                                <span>{{props.row.auditTime}}</span>
                            </el-form-item>

                            <el-form-item label="用户评论">
                                {{props.row.comment}}
                            </el-form-item>

                            <el-form-item label="活动总参与人数">
                                <span>{{props.row.count}}</span>
                            </el-form-item>

                            <el-form-item label="参与时间">
                                {{props.row.createTime}}
                            </el-form-item>

                            <el-form-item label="最大人数">
                                {{props.row.maxNumber}}
                            </el-form-item>


                            <el-form-item label="红包金额(元)">
                                {{props.row.money/100}}
                            </el-form-item>


                            <el-form-item label="用户昵称">
                                {{props.row.nickname}}
                            </el-form-item>

                            <el-form-item label="审核人ID">
                                {{props.row.operatorId}}
                            </el-form-item>

                            <el-form-item label="审核人名称">
                                {{props.row.operatorName}}
                            </el-form-item>

                            <el-form-item label="通过人数">
                                {{props.row.passNumber}}
                            </el-form-item>

                            <el-form-item label="红包已发数量">
                                {{props.row.redPackCount}}
                            </el-form-item>
                            <el-form-item label="红包发送时间">
                                {{props.row.redPacksendTime}}
                            </el-form-item>
                            <el-form-item label="拒绝原因"
                                          v-if="props.row.state=='01'?false:(props.row.state=='02'?false:true)">
                                {{props.row.rejectMessage}}
                            </el-form-item>

                            <el-form-item label="红包剩余数量">
                                {{props.row.remain}}
                            </el-form-item>

                            <el-form-item label="发送人名称">
                                {{props.row.sendOperatorName}}
                            </el-form-item>

                            <el-form-item label="审核状态">
                                {{props.row.state=='01'?'待审核':(props.row.state=='02'?'审核通过':'审核拒绝')}}
                            </el-form-item>

                            <el-form-item label="淘宝订单号">
                                {{props.row.taobaoOrder}}
                            </el-form-item>

                            <div style="width: 45%">
                                <el-input v-model="rejectMessage" type="textarea" placeholder="拒绝理由"></el-input>
                                <br>
                                <div style="margin-top: 2%;display: flex;flex-direction: row;justify-content: space-around">
                                    <el-button @click="reject(props.row.id)">拒绝</el-button>
                                    <el-button @click="through(props.row.id)">通过</el-button>
                                </div>

                            </div>

                        </el-form>
                    </template>
                </el-table-column>
                <el-table-column
                        type="selection"
                        align="center">
                </el-table-column>
                <el-table-column
                        prop="activityName"
                        label="活动名称"
                        width="180"
                        align="center"
                >
                </el-table-column>
                <el-table-column
                        prop="nickname"
                        label="粉丝昵称"
                        width="180"
                        align="center">
                </el-table-column>
                <el-table-column
                        label="红包金额(元)"
                        align="center">
                    <template slot-scope="scope">
                        {{scope.row.money/100}}
                    </template>
                </el-table-column>
                <el-table-column
                        label="状态"
                        align="center"
                        width="130">
                    <template slot-scope="scope">
                        {{scope.row.state=='01'?'待审核':(scope.row.state=='02'?'审核通过':'审核拒绝')}}
                    </template>

                </el-table-column>
                <el-table-column
                        prop="createTime"
                        label="参与时间"
                        align="center"
                ></el-table-column>
            </el-table>

            <div class="block">
                <el-pagination
                        layout="prev, pager, next"
                        :total="count"
                        :page-size="size"
                        @current-change="currChange"
                >
                </el-pagination>
            </div>

        </template>

    </div>
</template>

<script>
    export default {
        name: "audit",
        data() {
            return {
                acditList: [],//审核数据
                count: 0,//总条目数
                size: 0,//单页显示的数量
                rejectMessage: '',//拒绝原因
                acditId: [], //选中id
                actiName: '',//搜索活动名
                auditStartTime: '',//搜索开始时间
                auditEndTime: '',//搜索结束时间
                options: [{
                    value: '01',
                    label: '待审核'
                }, {
                    value: '02',
                    label: '审核通过'
                },{
                    value: '03',
                    label: '审核拒绝'
                }],
                value:''//选择的id
            }
        },
        methods: {
            //搜索活动
            search: function () {
                this.$http.get('http://jiajiachuang.cn/junran/manage/useractivity/search', {
                    headers: {token: this.$cookies.get('token')},
                    params: {size: 10,auditStartTime:this.auditStartTime,auditEndTime:this.auditEndTime,keyword:this.actiName,state:this.value}
                }).then(res => {
                    console.log(res.data)
                    if (res.data.code == 0) {
                        this.acditList = res.data.data
                        this.count = res.data.count
                        this.size = res.data.size

                    } else if (res.data.code == 103) {
                        alert('身份验证过期，请重新登录');
                        this.$router.push('./');
                    }
                })


            },
            //批量通过
            BatchThrough: function () {

                if (this.acditId.length == 0) {
                    alert('您还未选择数据！')
                } else {
                    this.$http.post('http://jiajiachuang.cn/junran/manage/useractivity/pass', {ids: this.acditId}, {
                        headers: {token: this.$cookies.get('token')}
                    }).then(res => {
                        if (res.data.code == 0) {
                            alert('操作成功！')
                            window.location.reload()
                        } else if (res.data.code == 103) {
                            alert('身份验证已过期，请重新登录');
                            this.$router.push('./')
                        } else {
                            alert(res.data.message)
                        }
                    })
                }


            },

            //将复选框选中id添加到数组
            selChange: function (val) {
                this.acditId.length = 0;
                for (var i = 0; i < val.length; i++) {
                    this.acditId.push(val[i].id)
                }
                console.log(this.acditId)
            },
            //活动通过
            through: function (id) {
                this.$http.post('http://jiajiachuang.cn/junran/manage/useractivity/pass', {ids: [id]}, {
                    headers: {token: this.$cookies.get('token')}
                }).then(res => {
                    if (res.data.code == 0) {
                        alert('操作成功！')
                        window.location.reload()
                    } else if (res.data.code == 103) {
                        alert('身份验证已过期，请重新登录');
                        this.$router.push('./')
                    } else {
                        alert(res.data.message)
                    }
                })
            },
            //活动拒绝
            reject: function (id) {
                this.$http.post('http://jiajiachuang.cn/junran/manage/useractivity/reject', {
                    id: id,
                    rejectMessage: this.rejectMessage
                }, {
                    headers: {token: this.$cookies.get('token')}
                }).then(res => {
                    if (res.data.code == 0) {
                        alert('操作成功！')
                        window.location.reload()
                    } else if (res.data.code == 103) {
                        alert('身份验证已过期，请重新登录');
                        this.$router.push('./')
                    }
                })
            },
            //分页
            currChange: function (val) {
                this.$http.get('http://jiajiachuang.cn/junran/manage/useractivity/search', {
                    headers: {token: this.$cookies.get('token')},
                    params: {size: 10, page: val - 1}
                }).then(res => {
                    if (res.data.code == 0) {
                        this.acditList = res.data.data
                        this.count = res.data.count
                        this.size = res.data.size

                    } else if (res.data.code == 103) {
                        alert('身份验证过期，请重新登录');
                        this.$router.push('./');
                    }
                })
            }
        },
        created: function () {
            this.$http.get('http://jiajiachuang.cn/junran/manage/useractivity/search', {
                headers: {token: this.$cookies.get('token')},
                params: {size: 10}
            }).then(res => {
                console.log(res.data)
                if (res.data.code == 0) {
                    this.acditList = res.data.data
                    this.count = res.data.count
                    this.size = res.data.size

                } else if (res.data.code == 103) {
                    alert('身份验证过期，请重新登录');
                    this.$router.push('./');
                }
            })


        }
    }
</script>

<style scoped>
    .audit {
        width: 100%
    }

    .demo-table-expand {
        font-size: 0;
    }

    .demo-table-expand label {
        width: 90px;
        color: #99a9bf;
    }

    .demo-table-expand .el-form-item {
        margin-right: 0;
        margin-bottom: 0;
        width: 50%;
    }
</style>