<!--系统设置-->
<template xmlns:v-on="http://www.w3.org/1999/xhtml">
    <div id="app" class="system">
        <router-view class="view" keep-alive></router-view>
    </div>
</template>
<script>
import { mapActions, mapState } from "vuex";
export default {
  data() {
    return {};
  },
  // 过滤
  filters: {},
  // 模块新建时
  created: function() {
    var self = this;
    self.fetchData();
  },
  // 计算属性
  mounted() {},
  // 事件处理
  methods: {
    // 路由监视
    fetchData() {
      var self = this;
      var paths = this.$route.path.split("/");
      var oneStr = paths[2];
      var twoStr = paths[3];
      var roleIdStr = sessionStorage.getItem("roleId");
      var TOKEN = sessionStorage.getItem("token");
      var userRoleOBj = sessionStorage.getItem("userRole");
      if (userRoleOBj) {
        userRoleOBj = JSON.parse(userRoleOBj);
        var arrs = [];
        var comms = [];
        userRoleOBj.forEach(function(ele, ind) {
          if (ele.name == "商城管理") {
            ele.children.forEach(function(one, oneIndex) {
              arrs.push(one.path);
              if (one.name == "系统设置") {
                one.children.forEach(function(two, twoIndex) {
                  comms.push(two.path);
                });
              }
            });
          }
        });
        if (arrs.indexOf("system") == -1) {
          var warningStr = "您暂时没有权限访问这个页面";
          self
            .$alert(warningStr, "提示", {
              confirmButtonText: "确定",
              type: "info"
            })
            .then(() => {
              self.$router.push({
                path: "/login"
              });
            })
            .catch(() => {
              self.$router.push({
                path: "/login"
              });
            });
        } else {
          if (twoStr == undefined || twoStr == "") {
            self.$router.push({
              path: "/store/system/" + comms[0]
            });
          } else {
            if (comms.indexOf(twoStr) == -1) {
            } else if (twoStr != comms[0]) {
              self.$router.push({
                path: "/store/system/" + twoStr
              });
            } else {
              self.$router.push({
                path: "/store/system/" + comms[0]
              });
            }
          }
        }
      } else {
        var warningStr = "没有登录不允许访问,点击确定跳转登录页面";
        this.$alert(warningStr, "提示", {
          confirmButtonText: "确定",
          type: "info"
        })
          .then(() => {
            this.$router.push({
              path: "/login"
            });
          })
          .catch(() => {
            this.$router.push({
              path: "/login"
            });
          });
      }
    },
    ...mapActions(["getListResRole"])
  },
  // 监视
  watch: {
    $route: "fetchData"
  }
};
</script>
<style scoped>
.view {
  width: 100%;
  height: 100%;
}
</style>