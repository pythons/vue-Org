<!--文件详情-->
<template xmlns:v-on="http://www.w3.org/1999/xhtml">
    <div id="app" class="sale">
        <div>
            <div class="listTitle">
                <div>
                    <div @click="canelClick">
                        <img src="/static/img/fan.png" alt="">
                    </div>
                    <div>
                        新增单页
                    </div>
                </div>
            </div>
            <!--列表-->
            <div class="list" v-loading="loading">
                <div class="list-code">
                    <div>
                        标题：
                    </div>
                    <div>
                        <input type="text" v-model="title">
                    </div>
                </div>
                <div class="list-code">
                    <div>
                        作者：
                    </div>
                    <div>
                        <input type="text" v-model="author">
                    </div>
                </div>
                <div class="list-code">
                    <div>
                        选择编码：
                    </div>
                    <div>
                        <select @change="selectcodesClick" v-model="codestr">
                                 <option v-for="(type,index) in codes" v-bind:value="type.dictionaryCode">{{type.dictionaryName}}</option>
                        </select>
                    </div>
                </div>
                <div class="list-cont">
                    <div id="editor" tabindex="-1" v-html="inputContent" style="width: 500px;height: 500px;">

                    </div>
                </div>
                <div class="but">
                    <div @click="addDoct">确定</div>
                    <div @click="canelClick">取消</div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import WangEditor from "wangeditor";
import { mapActions, mapState } from "vuex";
export default {
  components: {},
  data() {
    return {
      codes: [
        {
          dictionaryName: "请选择",
          dictionaryCode: "QINGXUNZE"
        }
      ],
      codestr: "QINGXUNZE",
      inputContent: "",
      title: "",
      author: "",
      addTrue: false,
      loading: false,
      content: "<p><br></p>"
    };
  },
  // 过滤
  filters: {},
  // 模块新建时
  created: function() {
    var self = this;
    // 权限限制
    var roleIdStr = sessionStorage.getItem("roleId");
    var TOKEN = sessionStorage.getItem("token");

    var userRoleOBj = sessionStorage.getItem("userRole");
    if (userRoleOBj) {
      userRoleOBj = JSON.parse(userRoleOBj);
      var arrs = [];
      userRoleOBj.forEach(function(ele, ind) {
        if (ele.name == "公共管理") {
          ele.children.forEach(function(one, oneIndex) {
            arrs.push(one.path);
          });
        }
      });
      if (arrs.indexOf("cut") == -1) {
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
        // 获取编码
        self
          .getbasefindListByParentId({
            parentId: 3440
          })
          .then(data => {
            // console.log("子字典列表", JSON.parse(JSON.stringify(data)));
            self.loading = false;
            if (data.code == 200) {
              if (data.data) {
                if (data.data.length > 0) {
                  data.data.forEach(function(ele) {
                    self.codes.push(ele);
                  });
                  self.addTrue = true;
                } else {
                  self.lists = [];
                }
              } else {
                self.lists = [];
              }
            } else {
              self.lists = [];
              self.$message({
                type: "info",
                message: data.message
              });
            }
          })
          .catch(msg => {
            self.loading = false;
            self.lists = [];
            // console.log(msg);
            self.$message({
              type: "info",
              message: "服务器异常"
            });
          });
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
  // 计算属性
  mounted() {
    this.createEditor();
  },
  // 事件处理
  methods: {
    // 编码选择
    selectcodesClick() {
      var self = this;
      if (self.codestr == "QINGXUNZE") {
        self.addTrue = true;
      } else {
        self
          .getcutpageContextselectPage({
            pageNo: 1,
            pageSize: 200,
            dictionaryCode: self.codestr
          })
          .then(data => {
            // console.log("单页列表", JSON.parse(JSON.stringify(data)));
            self.loading = false;
            if (data.code == 200) {
              if (data.data) {
                if (data.data.records) {
                  if (data.data.records.length > 0) {
                    self.$message({
                      type: "info",
                      message: "当前编码下已经存在单页，不能再进行添加"
                    });
                    self.addTrue = true;
                  } else {
                    self.addTrue = false;
                  }
                } else {
                  self.addTrue = false;
                }
              } else {
                self.addTrue = false;
              }
            } else {
              self.addTrue = false;
              self.$message({
                type: "info",
                message: data.message
              });
            }
          })
          .catch(msg => {
            self.addTrue = false;
            // console.log(msg);
            self.$message({
              type: "info",
              message: "服务器异常"
            });
          });
      }
    },
    // 箭头返回上一页/取消按钮
    canelClick() {
      this.$router.push({
        path: "/common/cut"
      });
    },
    // 编辑器处理
    createEditor() {
      const self = this;
      const editor = new WangEditor("editor");
      editor.config.menus = [
        "bold",
        "underline",
        "italic",
        "strikethrough",
        "eraser",
        "forecolor",
        "bgcolor",
        "|",
        "quote",
        "fontfamily",
        "fontsize",
        "head",
        "unorderlist",
        "orderlist",
        "alignleft",
        "aligncenter",
        "alignright",
        "|",
        "link",
        "unlink",
        "table",
        "img",
        "video",
        "insertcode",
        "|",
        "undo",
        "redo",
        "fullscreen"
      ];
      // 上传图片
      editor.config.uploadImgUrl = "/V1.0.0/api/file/uploadWebEditor";
      editor.config.menuFixed = false;
      var TOKEN = sessionStorage.getItem("token");
      editor.config.uploadParams = {
        appName: "BOTU",
        appId: "BOTU",
        module: "component",
        business: "singlePage",
        token: TOKEN
      };
      // 设置 headers（举例）
      editor.config.uploadHeaders = {
        Accept: "*"
      };
      // 隐藏掉插入网络图片功能。该配置，只有在你正确配置了图片上传功能之后才可用。
      //  editor.config.hideLinkImg = true;
      editor.onchange = function() {
        // console.log(this.$txt.html());
        self.content = this.$txt.html();
        self.inputText = this.$txt.text();
      };
      editor.create();
    },
    //确定新增
    addDoct() {
      var self = this;
      if (
        self.addTrue == false &&
        self.content.length != 11 &&
        self.content.length < 10000
      ) {
        self
          .getcutpageContextadd({
            dictionaryCodeJson: self.codestr,
            titleJson: self.title,
            authorJson: self.author,
            contentJson: self.content
          })
          .then(data => {
            // console.log("添加单页", JSON.parse(JSON.stringify(data)));
            self.loading = false;
            if (data.code == 200) {
              self.$router.push({
                path: "/common/cut"
              });
            } else {
              self.$message({
                type: "info",
                message: data.message
              });
            }
          })
          .catch(msg => {
            self.loading = false;
            // console.log(msg);
            self.$message({
              type: "info",
              message: "服务器异常"
            });
          });
      } else {
        if (self.codestr == "QINGXUNZE") {
          self.$message({
            type: "info",
            message: "请选择编码"
          });
        } else {
          self.$message({
            type: "info",
            message: "当前编码下已经存在单页，不能再进行添加"
          });
        }
        if (self.content.length == 11) {
          self.$message({
            type: "info",
            message: "最少一个文字或者一张图片"
          });
        }
        if (self.content.length > 10000) {
          self.$message({
            type: "info",
            message: "最多10000个字符"
          });
        }
      }
    },
    ...mapActions([
      "getbasefindListByParentId",
      "getcutpageContextadd",
      "getcutpageContextselectPage",
      "getListResRole"
    ])
  },
  // 监视
  watch: {}
};
</script>
<style scoped>
.sale {
  width: 100%;
  height: 100%;
}

.sale > div {
  overflow: hidden;
  width: calc(100% - 72px);
  margin-left: 36px;
}

.listTitle {
  height: 50px;
  overflow: hidden;
  margin-top: 10px;
  border-bottom: 1px solid #e5e5e5;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.listTitle > div {
  float: left;
  display: flex;
  align-items: center;
  justify-content: space-between;
  cursor: pointer;
}

.listTitle > div > div:nth-child(1) {
  width: 25px;
  height: 25px;
}

.listTitle > div > div:nth-child(1) img {
  margin: 0;
  padding: 0;
  width: 25px;
  height: 25px;
}

.listTitle > div > div:nth-child(2) {
  float: left;
  display: flex;
  color: #303030;
  font-size: 17px;
  margin-left: 10px;
}

.list {
  min-height: 540px;
  margin-top: 25px;
  overflow: hidden;
}

.list-code {
  display: flex;
  align-items: center;
  margin-bottom: 24px;
}

.list-code > div:nth-child(1) {
  min-width: 80px;
  text-align: right;
}

.list-code > div:nth-child(2) {
  margin-left: 15px;
}

.list-code input {
  width: 250px;
  height: 32px;
  border: 1px solid #d1d1d1;
  border-radius: 4px;
  padding-left: 5px;
}

.list-cont {
  margin-top: 54px;
  margin-bottom: 24px;
  overflow: hidden;
  min-width: 500px;
  height: 550px;
}

.list-cont .wangEditor-container {
  width: 500px !important;
}

.but {
  width: 100%;
  height: 80px;
  display: flex;
  justify-content: center;
  margin-bottom: 100px;
}

.list > div {
  width: 100%;
  overflow: hidden;
}

.list .but > div:nth-child(1) {
  width: 68px;
  height: 32px;
  border-radius: 3px;
  text-align: center;
  line-height: 32px;
  color: #fff;
  background-color: #fe5371;
  margin-right: 25px;
  cursor: pointer;
}

.list .but > div:nth-child(2) {
  margin-left: 25px;
  width: 68px;
  height: 32px;
  text-align: center;
  line-height: 32px;
  background-color: #aeaeae;
  color: #fff;
  border-radius: 3px;
  cursor: pointer;
}

select {
  width: 144px;
  height: 32px;
  line-height: 32px;
  padding-left: 7px;
  color: #8f8e94;
  border-radius: 3px;
  appearance: none;
  -moz-appearance: none;
  -webkit-appearance: none;
  background: url("/static/img/hjian.png") 90% 8px no-repeat transparent;
  /*padding-left: 14px;*/
  font-size: 14px;
  color: #97969b;
  border: 1px solid #d1d1d1;
  /*text-align: center;*/
  overflow: hidden;
}

select::-ms-expand {
  display: none;
}
</style>