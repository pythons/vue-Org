<!--标签管理-->
<template xmlns:v-on="http://www.w3.org/1999/xhtml">
    <div id="app" class="sale">
        <div>
            <div class="listTitle">
                <div>
                    标签管理
                </div>
                <div @click="addClick">
                    新建标签
                </div>
            </div>
            <!--列表-->
            <div class="list">
                <div class="listCont" v-loading="loading">
                    <ul>
                        <li class="firstLi">
                            <div class="firstCheckBox">
                                <el-checkbox v-model="checked" @change="numChange"></el-checkbox>
                                <div>
                                    <span>序号</span>
                                </div>
                            </div>
                            <div>
                                标签名称
                            </div>
                            <div>
                                操作
                            </div>
                        </li>
                        <li class="dataLi" v-show="lists.length > 0" v-for="(item,index) in lists" v-bind:key="index">
                            <div class="dataCheck">
                                <el-checkbox-group v-model="checkList" @change="handleCheckedCitiesChange">
                                    <el-checkbox :label="item" :key="item.id">{{checkNull}}</el-checkbox>
                                    <!-- <el-checkbox :label="item" :key="item.id">{{index + 1}}</el-checkbox> -->
                                </el-checkbox-group>
                                <div>
                                    {{index + 1}}
                                </div>
                            </div>
                            <div>
                                {{item.labelName |　strFun}}
                            </div>
                            <div class="operate">
                                <div @click="updateClick(index)"><img src="/static/img/编辑.png"></div>
                                <div @click="deleteClick(index)"><img src="/static/img/删除.png"></div>
                            </div>
                        </li>
                        <li class="nullList" v-show="lists.length == 0">
                            <div>
                                <img src="/static/img/w.png" alt="">
                            </div>
                        </li>
                        <li class="lastLi" v-show="lists.length > 0">
                            <div class="lastLiLeft">
                            </div>
                            <div id="lastLiRight">
                                <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :page-sizes="[10, 20, 30, 40]" :page-size="pageSizeNum"
                                    :current-page="currentPage" layout="total, sizes, prev, pager, next" :total="totalNum">
                                    </el-pagination>
                            </div>
                        </li>
                    </ul>
                </div>
                <div class="recycle" v-show="lists.length > 0">
                    <div @click="deletsClick">
                        删除
                    </div>
                </div>
            </div>
        </div>
        <!--添加弹出框-->
        <div :class="{addPopup:isAddPopup,showAdd:isShowAdd}">
            <transition name="slide-fade">
                <div class="addContent" v-show="addShow" v-loading="adding">
                    <div class="addTitle">
                    </div>
                    <div class="addNa">
                        <div>
                            标签：
                        </div>
                        <div>
                            <div>
                                <input id="addRoleInput" :class="{addInput:isaddattributeValue,grayInput:isRrayattributeValue}" maxlength="4" type="text"
                                    v-model="name">
                            </div>
                            <div :class="{addTest:isaddattributeValue,grayTest:isRrayattributeValue}">
                                可输入4个汉字
                            </div>
                        </div>
                    </div>
                    <div class="adBut">
                        <div @click="butAddClick">确定</div>
                        <div @click="cancelClcik">取消</div>
                    </div>
                </div>
            </transition>
        </div>
        <!--删除弹出框-->
        <div :class="{delPopup:isdelPopup,showdel:isShowdel}">
            <transition name="slide-fade">
                <div class="delContent" v-show="delShow" v-loading="deling">
                    <v-storeDel @del="delClick" @canle="cancelClcik" :name="titlename"></v-storeDel>
                </div>
            </transition>
        </div>
    </div>
</template>
<script>
import { mapActions, mapState } from "vuex";
import storeDel from "../../cModule/finessDel";
export default {
  components: {
    "v-storeDel": storeDel
  },
  data() {
    return {
      checked: false,
      checkedAll: true,
      loading: false,
      checkNull: "",
      checkList: [],
      lists: [],
      pageSizeNum: 10,
      currentPage: 1,
      totalNum: 0,
      deling: false,
      adding: false,
      name: "",
      isRrayattributeValue: true,
      isaddattributeValue: false,
      isattributeValue: false,
      isAddPopup: true,
      isShowAdd: true,
      addShow: false,
      upOrAd: 1,
      upIndex: 0,
      titlename: "",
      isdelPopup: true,
      isShowdel: true,
      delShow: false,
      delIndex: 0,
      idArr: [],
      delClickTrue: false,
      butAddClickTrue: false,
    };
  },
  // filters
  filters: {
    strFun: function(val) {
      if (val) {
        return val;
      } else {
        return "无";
      }
    }
  },
  // initialize function
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
        if (ele.name == "健身管理") {
          ele.children.forEach(function(one, oneIndex) {
            arrs.push(one.path);
          });
        }
      });
      if (arrs.indexOf("tally") == -1) {
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
        self.listFun();
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
  // count property
  mounted() {},
  // incident function
  methods: {
    // initialize data
    listFun() {
      var self = this;
      self.loading = true;
      self.checkList = [];
      self.checked = false
      self
        .getfitnessLabellControllerselectLabellPage({
          current: self.currentPage,
          size: self.pageSizeNum
        })
        .then(data => {
          self.loading = false;
          //console.log("列表", JSON.parse(JSON.stringify(data)));
          if (data.code == 200) {
            if (data.data) {
              if (data.data.records) {
                if (data.data.records.length > 0) {
                  self.lists = data.data.records;
                  self.totalNum = data.data.total;
                  // self.lists.forEach(function(ele,ind){        
                  //   ele["checked"] = false      
                  // })  
                  //console.log(self.lists)
                } else {
                  self.lists = [];
                }
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
          self.lists = [];
          //console.log(msg);
          self.loading = false;
          self.$message({
            type: "info",
            message: "服务器异常!"
          });
        });
    },
    // batch delete
    deletsClick() {
      var self = this;
      self.idArr = [];
      if (this.checkList.length > 0) {
        this.checkList.forEach(function(ele, index) {
          self.idArr.push(ele.id);
        });
        this.isShowdel = false;
        this.delShow = true;
        this.titlename = "标签";
      } else {
        self.$message({
          type: "info",
          message: "请选择标签!"
        });
      }
    },
    numChange(event) {
      var self = this
      self.checkList = event.target.checked ? self.lists : [];
    },
    // click select number
    handleCheckedCitiesChange(value) {
      var self = this
      let checkedCount = value.length;
      self.checked = checkedCount === self.lists.length;
    },
    // paging-pageNumber
    handleSizeChange(val) {
      var self = this;
      self.pageSizeNum = val;
      self.listFun();
    },
    //  paging-pageSize
    handleCurrentChange(val) {
      var self = this;
      self.currentPage = val;
      self.listFun();
    },
    // compile
    updateClick(index) {
      this.isShowAdd = false;
      this.addShow = true;
      this.name = this.lists[index].labelName;
      this.upOrAd = 2;
      this.upIndex = index;
    },
    // addtion
    addClick() {
      this.isShowAdd = false;
      this.addShow = true;
      this.upOrAd = 1;
    },
    validateFun() {
      var reg=/^[\u2E80-\u9FFF]+$/;//Unicode编码中的汉字范围
      if (!reg.test(this.name)) {
        this.isaddattributeValue = true
        this.isRrayattributeValue = false

      } else {
        this.isaddattributeValue = false
        this.isRrayattributeValue = true
      }
      if (this.isaddattributeValue == true) {
        return false
      }
    },
    // 确认添加或者修改
    butAddClick() {
      var self = this;
      if (self.validateFun() == false) {
        return 
      } else {
         if (self.butAddClickTrue == false) {
        if (self.isattributeValue == true) {
          if (self.upOrAd == 1) {
            self.adding = true;
            self.butAddClickTrue = true;
            self.getfitnessLabellControllersave({
                name: self.name
              })
              .then(data => {
                self.adding = false;
                self.isShowAdd = true;
                self.addShow = false;
                self.name = "";
                self.butAddClickTrue = false;
                if (data.code == 200) {
                  self.$message({
                    type: "info",
                    message: "添加成功!"
                  });
                  self.listFun();
                } else {
                  self.$message({
                    type: "info",
                    message: data.message
                  });
                }
              })
              .catch(msg => {
                self.butAddClickTrue = false;
                self.adding = false;
                self.$message({
                  type: "info",
                  message: "服务器异常!"
                });
              });
          } else {
            self.adding = true;
            self.butAddClickTrue = true;
            self
              .getfitnessLabellControllerupdateLabel({
                id: self.lists[self.upIndex].id,
                name: self.name
              })
              .then(dataThr => {
                self.adding = false;
                self.butAddClickTrue = false;
                if (dataThr.code == 200) {
                  self.$message({
                    type: "info",
                    message: "修改成功!"
                  });
                  self.name = "";
                  self.isShowAdd = true;
                  self.addShow = false;
                  self.listFun();
                } else {
                  self.$message({
                    type: "info",
                    message: data.message
                  });
                }
              })
              .catch(msg => {
                self.butAddClickTrue = false;
                self.adding = false;
                self.$message({
                  type: "info",
                  message: "服务器异常!"
                });
              });
          }
        } else {
          if (self.isattributeValue == false) {
            self.isRrayattributeValue = false;
            self.isaddattributeValue = true;
          } else {
            self.isRrayattributeValue = true;
            self.isaddattributeValue = false;
          }
        }
      }
      }
     
    },
    // delete
    deleteClick(index) {
      this.isShowdel = false;
      this.delShow = true;
      this.delIndex = index;
      this.titlename = "标签";
    },
    // affirm delete
    delClick() {
      var self = this;
      if (self.delClickTrue == false) {
        self.deling = true;
        var idStr = "";
        if (self.idArr.length > 0) {
          if (self.idArr.length == 1) {
            idStr = '["' + self.idArr[0] + '"]';
          } else {
            idStr = idStr = JSON.stringify(self.idArr);
          }
        } else {
          idStr = '["' + self.lists[self.delIndex].id + '"]';
        }
        self.delClickTrue = true;
        self
          .getfitnessLabellControllerdeleteById({
            ids: idStr
          })
          .then(data => {
            self.deling = false;
            self.checked = false;
            self.checkList = [];
            self.isShowdel = true;
            self.delShow = false;
            self.delClickTrue = false;
            if (data.code == 200) {
              self.$message({
                type: "success",
                message: "删除成功!"
              });
              if (self.idArr.length > 0) {
                var currNum = self.currentPage;
                if (
                  self.idArr.length == self.lists.length &&
                  self.currentPage > 1
                ) {
                  self.currentPage = currNum - 1;
                }
              } else {
                var currNum = self.currentPage;
                if (self.lists.length == 1 && self.currentPage > 1) {
                  self.currentPage = currNum - 1;
                }
              }
              self.listFun();
              self.idArr = [];
            } else {
              self.$message({
                type: "info",
                message: data.message
              });
            }
          })
          .catch(msg => {
            self.checked = false;
            self.checkList = [];
            self.idArr = [];
            self.isShowdel = true;
            self.delShow = false;
            self.delClickTrue = false;
            self.deling = false;
            self.$message({
              type: "info",
              message: "服务器异常!"
            });
          });
      }
    },
    // cancel delete
    cancelClcik() {
      this.isShowdel = true;
      this.delShow = false;
      this.titlename = "";
      this.isShowAdd = true;
      this.addShow = false;
      this.name = "";
    },
    ...mapActions([
      "getfitnessLabellControllersave",
      "getfitnessLabellControllerdeleteById",
      "getfitnessLabellControllerselectLabellPage",
      "getfitnessLabellControllerupdateLabel"
    ])
  },
  // watch model
  watch: {
    name: function(val) {
      var trimstr = val;
      var reg = new RegExp(/[\u4e00-\u9fa5]{1,5}$/g);
      if (trimstr.length <= 0) {
        this.isattributeValue = false;
      } else {
        if (trimstr.length < 5) {
          this.isRrayattributeValue = true;
          this.isaddattributeValue = false;
          this.isattributeValue = true;
        } else {
          this.isattributeValue = false;
        }
      }
    }
  }
};
</script>
<style scoped>
.list .nullList {
  height: 400px;
}

.list .nullList > div:nth-child(1) {
  width: 607px;
  height: 400px;
  margin: 0 auto;
  border: 0;
  overflow: hidden;
}

.sale {
  width: 100%;
  height: 100%;
}

.sale > div {
  overflow: hidden;
  width: calc(100% - 72px);
  margin-left: 36px;
}

.recycle {
  height: 100px;
  width: 100%;
  overflow: hidden;
  display: flex;
}

.recycle > div:nth-child(1) {
  margin-top: 3px;
  margin-left: 25px;
  width: 70px;
  height: 30px;
  background-color: #5acdfa;
  color: #fff;
  font-size: 16px;
  border-radius: 3px;
  line-height: 32px;
  cursor: pointer;
  text-align: center;
}

#lastLiRight {
  width: 80%;
  justify-content: flex-end;
  display: flex;
  align-items: center;
  margin-right: 20px;
}

.lastLi > div:nth-child(1) {
  width: 10px;
}

.lastLi > div:nth-child(2) {
  width: 80%;
  display: flex;
  justify-content: flex-end;
}

.lastLi {
  height: 60px;
  display: flex;
  justify-content: space-between;
}

.operate {
  align-items: center;
  display: flex;
}

.operate > div {
  width: 30px;
  height: 50px;
  cursor: pointer;
  overflow: hidden;
}

.operate > div:nth-child(1) {
  margin-right: 9px;
}

.operate > div:nth-child(2) {
  margin-left: 9px;
}

.operate > div img {
  width: 25px;
  height: 25px;
}

.listCont {
  margin-top: 36px;
}

.listCont li {
  display: flex;
  min-height: 50px;
}

.firstLi {
  border-top: 1px solid #d9d9d9;
  line-height: 58px;
  height: 58px;
  background-color: #f2f4f8;
  border-bottom: 1px solid #d9d9d9;
}

.dataCheck > div:nth-child(1) {
  width: 20px;
  height: 20px;
  line-height: 20px;
}

.dataCheck > div:nth-child(1) img {
  width: 20px;
  height: 20px;
}

.dataCheck > div:nth-child(2) {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 22px;
}

.dataCheck > div:nth-child(2) {
  width: 25px;
  line-height: 22px;
  height: 25px;
}

.dataCheck > div:nth-child(3) {
  margin-left: 5px;
}

.dataCheck > div:nth-child(3) span {
  height: 40px;
  margin-top: 1px;
  display: block;
  font-size: 15px;
}

.firstCheckBox {
  display: flex;
  align-items: center;
  justify-content: center;
}

.firstCheckBox > div:nth-child(1) {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 0px;
  width: 20px;
  height: 20px;
}

.firstCheckBox > div:nth-child(1) img {
  width: 20px;
  height: 20px;
}

.firstCheckBox > div:nth-child(2) {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-left: 8px;
}

.firstLi > div {
  border-right: 1px solid #e5e7eb;
  text-align: center;
  color: #303030;
  font-size: 17px;
  font-weight: bold;
}

.firstLi > div:nth-child(1) {
  border-left: 1px solid #e5e7eb;
}

.dataLi {
  border-bottom: 1px solid #ececec;
  background-color: #fff;
  height: 50px;
  line-height: 50px;
}

.listCont .dataLi > div {
  border-right: 1px solid #ececec;
  font-size: 14px;
  color: #515151;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  padding-left: 10px;
  padding-right: 10px;
}

.listCont .firstLi > div {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  padding-left: 10px;
  padding-right: 10px;
}

.dataLi > div:nth-child(2) {
  font-size: 14px;
  cursor: pointer;
  text-align: center;
}

.dataLi > div:nth-child(1) {
  border-left: 1px solid #ececec;
}

.listCont li > div:nth-child(1) {
  width: 33.33333333%;
  overflow: hidden;
  min-width: 70px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.listCont li > div:nth-child(2) {
  width: 33.33333333%;
  min-width: 100px;
}

.listCont li > div:nth-child(3) {
  width: 33.33333333%;
  min-width: 165px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.list {
  min-height: 540px;
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

.listTitle > div:nth-child(1) {
  float: left;
  color: #303030;
  font-size: 16px;
  font-weight: 600;
}

.listTitle > div:nth-child(2) {
  float: right;
  width: 80px;
  height: 26px;
  border: 1px solid #5acdfa;
  border-radius: 3px;
  text-align: center;
  line-height: 24px;
  color: #5acdfa;
  cursor: pointer;
}

select {
  width: 120px;
  height: 32px;
  line-height: 32px;
  padding-left: 7px;
  color: #8f8e94;
  border-radius: 3px;
  -moz-appearance: none;
  -webkit-appearance: none;
  background: url("/static/img/hjian.png") 90% 10px no-repeat transparent;
  font-size: 14px;
  color: #97969b;
  border: 1px solid #d1d1d1;
  overflow: hidden;
}

select::-ms-expand {
  display: none;
}

.addTest {
  color: #fe5371;
}

.grayTest {
  color: #d1d1d1;
}

.addPopup {
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100%;
  background-color: rgba(0, 0, 0, 0);
  z-index: 1000000;
  display: flex;
  align-items: center;
  justify-content: center;
}

.addContent > div:nth-child(1) {
  height: 15px;
  width: 100%;
}

.addContent input {
  width: 270px;
  height: 32px;
  border-radius: 4px;
  color: #303030;
  text-align: center;
  border: 1px solid #d1d1d1;
}

.addContent {
  width: 450px;
  min-height: 100px;
  background-color: #fff;
  border: 1px solid #d1d1d1;
  border-radius: 7px;
  z-index: 1;
  padding-bottom: 25px;
}

.showAdd {
  display: none;
}

.slide-fade-enter-active {
  transition: all 0.7s ease;
}

.slide-fade-enter,
.slide-fade-leave-active {
  transform: translateY(-30px);
  opacity: 0;
}

.adBut {
  width: 100%;
  height: 35px;
  overflow: hidden;
  display: flex;
  justify-content: center;
  margin-top: 15px;
}

.adBut > div {
  width: 68px;
  height: 30px;
  cursor: pointer;
}

.adBut > div:nth-child(1) {
  width: 68px;
  height: 30px;
  background-color: #5acdfa;
  text-align: center;
  line-height: 30px;
  border-radius: 3px;
  color: #fff;
  margin-right: 20px;
}

.adBut > div:nth-child(2) {
  width: 68px;
  height: 30px;
  border-radius: 3px;
  color: #fff;
  margin-left: 25px;
  background-color: #d1d1d1;
  text-align: center;
  line-height: 30px;
}

.addNa {
  display: flex;
  align-items: center;
  min-height: 55px;
  width: 100%;
  margin-top: 20px;
}

.addNa > div:nth-child(2) {
  margin-left: 15px;
  /*border: 1px solid red;*/
  height: 70px;
  overflow: hidden;
}

.addNa > div:nth-child(2) > div {
  width: 280px;
  overflow: hidden;
}

.addNa > div:nth-child(2) > div:nth-child(2) {
  margin-top: 8px;
}

.addNa > div:nth-child(1) {
  width: 105px;
  height: 70px;
  text-align: right;
  font-size: 15px;
  color: #303030;
  padding-top: 5px;
}
/*删除弹出框*/

.delContent {
  width: 525px;
  height: 280px;
  background-color: #fff;
  border: 1px solid #eaeaea;
  box-shadow: 0px 10px 10px #eaeaea;
  border-radius: 7px;
  margin: calc((100vh - 280px) / 2) auto;
  z-index: 100000;
}

.delPopup {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0);
  z-index: 1000000;
}

.showdel {
  display: none;
}
</style>