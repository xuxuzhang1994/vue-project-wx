
<template>
    <div class="statistics-app-detail" v-if='activeData.length>0'>
        <!--<div class='header flex' style='justify-content: center'>-->
        <!--<div class='title' >App分时段日活详情</div>-->
        <!--</div>-->
        <!--<div class='tab flex'>-->
        <!--<div class='tab-item tab-active'>日活</div>-->
        <!--<div class='tab-item'>周活</div>-->
        <!--<div class='tab-item'>月活</div>-->
        <!--</div>-->
        <div class='search-bar'>
            <div class='bar flex'>
                <div class='left flex'>
                    <div class='text'>当前显示:</div>
                    <div class='current-params'>
                        <span>{{params.stime*1000 | parseDate('YYYY.M.D') }}~{{ params.etime *1000 |
                            parseDate('YYYY.M.D')}}</span>
                        <span v-show='params.type == 1'>天</span>
                        <span v-show='params.type == 2'>周</span>
                        <span v-show='params.type == 3'>月</span>
                    </div>
                </div>
                <div class='right flex' @click='search()'>
                    <i class='icon anticon icon-filter'></i>
                    <span>筛选</span>
                </div>
            </div>
            <div class='content' v-if='searchBoxStatus'>
                <div class='params-item'>
                    <p>筛选单位</p>
                    <div class='type-box flex'>
                        <span :class="{'type-active' : type == 1}" @click='chooseType(1)'>天</span>
                        <span :class="{'type-active' : type == 2}" @click='chooseType(2)'>周</span>
                        <span :class="{'type-active' : type == 3}" @click='chooseType(3)'>月</span>
                    </div>
                </div>
                <div class='params-item' v-show='type == 1'>
                    <p>开始时间</p>
                    <div class='type-box flex'>
                        <input type="date" v-model='startTime'>
                        <span class='type-active'>{{startTime | parseDate('YYYY.M.D')}}</span>
                    </div>
                </div>
                <div class='params-item' v-show='type == 1'>
                    <p>结束时间</p>
                    <div class='type-box flex'>
                        <input type="date" v-model='endTime'>
                        <span class='type-active'>{{endTime | parseDate('YYYY.M.D')}}</span>
                    </div>
                </div>
                <div class='confirm' @click='confirm()'>确认</div>
            </div>

        </div>
        <div class='mask' v-show='searchBoxStatus'></div>
        <div class='data-list' :class='{"filter": searchBoxStatus}'>
            <div class='data-item'
                 @click='deploy(item)'
                 v-for='(item,$index) in activeData'>
                <div class='preview flex'>
                    <div class='left'>
                        <i class='icon iconfont icon-rili'  :class="iconClass($index,item)"
                           v-show='params.type == 1'></i>
                        <img src="../../../../images/statistics/周1.png" alt="" v-show='params.type == 2'>
                        <img src="../../../../images/statistics/月1.png" alt="" v-show='params.type == 3'>
                    </div>
                    <div class='right flex '>
                        <div class='time' v-show='params.type == 1' v-text='item.parseDay'>2018.5.10</div>
                        <div class='time' v-show='params.type == 2'>第 {{Math.abs($index - activeData.length)}} 周</div>
                        <div class='time' v-show='params.type == 3'>{{item.day*1000 | parseDate('YYYY年M月')}}</div>
                        <div class='user-total'>
                            <span>总计: </span>
                            <span>{{item.total}}</span>
                        </div>
                    </div>
                    <div class='more flex'>
                        <i class='iconfont icon-arrowB' :class='{"translate-180":item.detail}'></i>
                    </div>
                </div>
                <div class='detail' v-bind:class='{"detail-show":item.detail}'>
                    <div class='detail-item flex'>
                        <span>iPhone</span>
                        <span v-text='item.iphone'></span>
                    </div>
                    <div class='detail-item flex'>
                        <span>Android</span>
                        <span v-text='item.anzhuo'></span>
                    </div>
                    <div class='detail-item flex'>
                        <span>iPad</span>
                        <span v-text='item.ipad'></span>
                    </div>
                </div>
            </div>
        </div>
        <!--<div class='card-box'>-->
        <!--<div class='card-list'>-->
        <!--<div class='card-item flex'-->
        <!--v-for='(item) in loginList.dataList'-->
        <!--@click='goDetial(item)'>-->
        <!--<div class='time flex' v-text='item.parseDay'></div>-->
        <!--<div class='person-list'>-->
        <!--<div class='person-item flex'>-->
        <!--<span class='gray'>iphone：</span><span v-text='item.iphone'>20</span>-->
        <!--</div>-->
        <!--<div class='person-item flex'>-->
        <!--<span class='gray'>android：</span>-->
        <!--<span v-text='item.anzhuo'>17</span>-->
        <!--</div>-->
        <!--<div class='person-item flex'>-->
        <!--<span class='gray'>ipad：</span><span v-text='item.ipad'>13</span>-->

        <!--</div>-->
        <!--</div>-->
        <!--<div class='total'>-->
        <!--<div>总计</div>-->
        <!--<div class='total-number' v-text='item.total'>50</div>-->
        <!--</div>-->
        <!--</div>-->
        <!--</div>-->
        <!--</div>-->
    </div>
</template>

<script>
    import api from "../../../api/statistics-api";
    export default {
        name: 'statistics222',
        data: function () {
            return {
                searchBoxStatus:false,
                show:false,
                type: 1,
                startTime: moment(moment(new Date()).format('YYYY-MM')).format('x')/1,
                endTime: moment(moment(new Date()).format('YYYY-MM-DD')).format('x')/1,
                params:{
                    type: 1,
                    // 默认展示当月数据
                    stime: moment(moment(new Date()).format('YYYY-MM')).format('x')/1000,
                    etime: moment(moment(new Date()).format('YYYY-MM-DD')).format('x')/1000
                },
                cardData:{},
                activeData: [],
                loadData:true,
                iconClass:function (index,item) {
                    if(item.detail){
                        return {
                            ['icon-rili'+parseInt(item.parseDay[2] + item.parseDay[3] -1)]:true
                        }
                    }else {
                        return {
                            ['icon-rili'+parseInt(item.parseDay[2] + item.parseDay[3] -1)]:true
                        }
                    }


                },
                arr:[{detail:false},{detail:false},{detail:false}]
            }
        },
        created() {
            this.page.setTitle("app新增用户统计");
            this.getLoginData()
        },
        methods: {
            chooseType(type) {
                console.log(type)
                var self = this;
                self.type = type
            },
            search(){
                var self = this;
                self.searchBoxStatus = !self.searchBoxStatus
            },
            deploy(item){
                var self = this
                if(item.detail){
                    self.arr.forEach(e => {
                        e.detail = false
                    })
                    item.detail = false
                }else {
                    self.arr.forEach(e => {
                        e.detail = false
                    })
                    item.detail = true
                }
            },
            confirm(){
                var self = this;
                self.searchBoxStatus = false
                let toast=this.$toast;
                if(moment(self.startTime).format('x')/1000 > moment(self.endTime).format('x')/1000){
                    toast.show({
                        showTime: 2,
                        message: '结束时间不能早于开始时间',
                        style:'error'
                    });
                    return false
                }
                self.getLoginData()
            },
            goDetial(e){
                this.$router.push({path:'/statistics/statistics-active-hour-detail',query: { day : e.day}});
            },
            getLoginData(){
                console.log(moment(moment(new Date()).format('YYYY-MM')).format('x'))
                let self = this
                let loading=this.$loading
                let toast=this.$toast;
                self.loadData = true;
                loading.show('加载中...')
                self.params.type = self.type
                if(self.params.type == 1){
                    self.params.stime = moment(self.startTime).format('x')/1000
                    self.params.etime = moment(self.endTime).format('x')/1000
                }else {
                    self.params.stime = moment(moment(new Date()).format('YYYY')).format('x')/1000
                    self.params.etime = moment(moment(new Date()).format('YYYY-MM-DD')).format('x')/1000
                }
                api.app_login(self.params).then(function (data) {
                    if(data.code == 200){
                        data.data.forEach(e => {
                            e.parseDay = moment(e.day*1000).format('M月DD日')
                            e.detail = false
                        })
                        self.activeData = data.data
                        console.log(self.params)
                    }else{
                        toast.show({
                            showTime: 2,
                            message: data.msg,
                            style:'error'
                        });
                        loading.hide()
                    }
                    self.loadData = false;
                    loading.hide()
                })
                    .catch(function (error) {
                        console.log(error);
                    });
            }
        },

    }
</script>

<style lang="less" scoped>
    @import "./statistics-app-detail.less";
</style>