<!--评论管理-->
<template xmlns:v-on="http://www.w3.org/1999/xhtml">
    <div id="app" class="sale">
        <div>
            <div class="listTitle">
                <div>
                    评论管理
                </div>
                <div @click="addCommClick">
                    <!--新增热词-->
                </div>
            </div>
            <!--列表-->
            <div class="list">
                <div>
                    <img src="/static/img/缺省页1.png" alt="">
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import { mapActions, mapState } from 'vuex'
    export default {
        components: {
        },
        data() {
            return {
            }
        },
        // 过滤
        filters: {
        },
        // 模块新建时
        created: function () {
            var self = this
            // 权限限制
            var roleIdStr = sessionStorage.getItem('roleId')
            var TOKEN = sessionStorage.getItem('token')

            var userRoleOBj = sessionStorage.getItem('userRole')
            if (userRoleOBj) {
                userRoleOBj = JSON.parse(userRoleOBj)
                var arrs = []
                var arrs = []
                userRoleOBj.forEach(function (ele, ind) {
                    if (ele.name == "公共管理") {
                        ele.children.forEach(function (one, oneIndex) {
                            arrs.push(one.path)
                        })
                    }
                })
                if (arrs.indexOf("comment") == -1) {
                    var warningStr = "您暂时没有权限访问这个页面"
                    self.$alert(warningStr, '提示', {
                        confirmButtonText: '确定',
                        type: 'info'
                    }).then(() => {
                        self.$router.push({
                            path: '/login'
                        })
                    }).catch(() => {
                        self.$router.push({
                            path: '/login'
                        })
                    });
                }
            } else {
                var warningStr = "没有登录不允许访问,点击确定跳转登录页面"
                this.$alert(warningStr, '提示', {
                    confirmButtonText: '确定',
                    type: 'info'
                }).then(() => {
                    this.$router.push({
                        path: '/login'
                    })
                }).catch(() => {
                    this.$router.push({
                        path: '/login'
                    })
                });
            }
        },
        // 计算属性
        mounted() {

        },
        // 事件处理
        methods: {
            addCommClick() {
            },
            ...mapActions(['getListResRole'])
        }
    }

</script>
<style scoped>
    /*主页*/
    
    .sale {
        width: 100%;
        height: 100%;
    }
    
    .sale>div:nth-child(1) {
        overflow: hidden;
        width: calc(100% - 72px);
        margin-left: 36px;
    }
    
    .list {
        min-height: 540px;
    }
    
    .list>div {
        margin: 0 auto;
        width: 650px;
    }
    
    .listTitle {
        height: 58px;
        line-height: 58px;
        overflow: hidden;
        border-bottom: 1px solid #e5e5e5;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    
    .listTitle>div:nth-child(1) {
        float: left;
        color: #303030;
        font-size: 16px;
        font-weight: 600;
    }
    
    .listTitle>div:nth-child(2) {
        float: right;
        width: 80px;
        height: 26px;
        border: 1px solid #fff;
        border-radius: 3px;
        text-align: center;
        line-height: 24px;
        color: #fe5371;
        cursor: pointer;
    }
</style>