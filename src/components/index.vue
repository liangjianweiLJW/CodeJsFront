<template>
  <div>
    <div class="question">
      <el-row type="flex" justify="center">
        <el-col :xs="12" :sm="16" :md="14" :lg="12" :xl="10">
          <div class="question-simple">
            <div class="question-simple-q">
              <p>Q :)</p>
              <div class="question-simple-name" v-text="randomQuestion.name"></div>
            </div>
            <hr style="border-top: 1px solid #3c3c3c; margin-bottom: 10px;">
            <div class="work-button">
              <button
                @click="go()"
                style="background-color:#3c7dba;width:80px;border-radius:6px;border-color:#3c7dba;color:#303133;font-size:20px"
              >try</button>
              <button
                @click="skip()"
                style="background-color:rgba(0,0,0,0.2);width:80px;border-radius:6px;border-color:#3c7dba;color:#4467ab;font-size:20px"
              >skip</button>
            </div>
          </div>
        </el-col>
        <el-col :xs="10" :sm="8" :md="8" :lg="6" :xl="5">
          <div class="question-des">
            <div class="question-des-body">
              <p>D :)</p>
              <div class="question-des-main" v-text="randomQuestion.description"></div>
            </div>
            <hr style="border-top: 1px solid #262729;margin-bottom: 10px;">
            <div class="tip">
              <p>
                TIP: 本题
                <span v-text="randomQuestion.score"></span>分
              </p>
            </div>
          </div>
        </el-col>
      </el-row>
    </div>
    <div>
      <div class="team">
        <el-row type="flex" justify="center">
          <el-col :xs="22" :sm="24" :md="22" :lg="18" :xl="15">
            <div v-if="haveTeam" class="team-main">
              <div class="team-member">
                <el-row type="flex" justify="space-between">
                  <el-col :xs="22" :sm="24" :md="22" :lg="18" :xl="15">排名</el-col>
                  <el-col :xs="22" :sm="24" :md="22" :lg="18" :xl="15">成员</el-col>
                  <el-col :xs="22" :sm="24" :md="22" :lg="18" :xl="15">分数</el-col>
                  <el-col :xs="22" :sm="24" :md="22" :lg="18" :xl="15">空间</el-col>
                </el-row>
              </div>
              <div v-for="(item, index) in teamall" :key="item.id" class="team-member">
                <hr style="border-top: 1px solid #221f1f; margin-bottom: 30px;margin-top: -5px;">
                <MemberItem class="team-member-info" :teamMember="item" :index="index + 1"></MemberItem>
              </div>
            </div>
            <div v-else class="team-main">
              <p class="team-msg">{{teamMsg}}</p>
              <input
                v-if="show && !join"
                type="button"
                value="创建团队"
                class="login"
                @click="createTeamBtn()"
              >
              <input
                v-if="show && !join"
                type="button"
                value="加入团队"
                class="login"
                @click="joinTeamBtn()"
              >
              <div v-if="!show">
                <el-row type="flex" justify="center" class="body">
                  <el-col :span="12">
                    <input
                      style="color:black"
                      type="text"
                      name="teamName"
                      class="teamName"
                      id="teamName"
                      placeholder="请输入团队名称"
                    >
                  </el-col>
                </el-row>
                <el-row type="flex" justify="space-around" class="body">
                  <el-col :span="24">
                    <input
                      type="button"
                      @click="createTeam()"
                      name="login"
                      class="login"
                      value="创建"
                    >
                    <input
                      type="button"
                      @click="cancelCreate()"
                      name="login"
                      class="login"
                      value="取消"
                    >
                  </el-col>
                </el-row>
              </div>
              <div v-if="join">
                <el-row type="flex" justify="center" class="body">
                  <el-col :span="12">
                    <input
                      style="color:black"
                      type="text"
                      name="teamName"
                      class="teamName"
                      id="joinTeamName"
                      placeholder="请输入团队名称"
                    >
                  </el-col>
                </el-row>
                <el-row type="flex" justify="space-around" class="body">
                  <el-col :span="24">
                    <input type="button" @click="joinTeam()" name="login" class="login" value="加入">
                    <input
                      type="button"
                      @click="cancelJoin()"
                      name="login"
                      class="login"
                      value="取消"
                    >
                  </el-col>
                </el-row>
              </div>
            </div>
          </el-col>
        </el-row>
      </div>
    </div>
  </div>
</template>

<script>
import NProgress from "NProgress";
import * as types from "@/store/types";
import MemberItem from "@/components/base/teamItem/member-item";
export default {
  name: "Index",
  data() {
    return {
      randomQuestion: {},
      user: {},
      teamInfo: {},
      haveTeam: false,
      show: true,
      teamMsg: "抱歉您还没有参加或创建团队!",
      teamall: [],
      join: false
    };
  },
  methods: {
    skip() {
      this.$ajax({
        method: "post",
        url: "/api/getRandomQuestion?mobile=" + this.user.mobile
      }).then(res => {
        if (res.data.length == 0) {
          this.$ajax({
            url: "/otherApi/all.json",
            method: "post"
          }).then(res => {
            var content = res.data.content;
            var author = res.data.author;
            this.goodjob = content;
            this.randomQuestion = {name: content,description: author,id: null}
          })
        } else {
          this.randomQuestion = res.data;
        } 
      });
    },
    go() {
      if (this.randomQuestion.id != null) {
        this.$router.push("/work?id=" + this.randomQuestion.id);
      } else {
            floatMessage("没题目了,不如赏赏诗吧~")
            $(".trigger-info").click()
      }
    },
    createTeam() {
      var teamName = $("#teamName").val();
      this.$ajax({
        method: "post",
        url: "/api/maketeam",
        data: {
          teamName: teamName,
          mobile: this.user.mobile
        }
      }).then(res => {
        if (res.data == "小队创建成功!") {
          location.reload();
        } else {
           floatMessage(res.data)
           $(".trigger-info").click()
        }
      });
    },
    createTeamBtn() {
      this.show = false;
      this.teamMsg = "开始小队之旅吧~"
    },
    cancelCreate() {
      this.show = true;
      this.teamMsg = "抱歉您还没有参加或创建团队!"
    },
    cancelJoin() {
      this.join = false;
      this.teamMsg = "抱歉您还没有参加或创建团队!";
    },
    joinTeamBtn() {
      this.teamMsg = "开始小队之旅吧~";
      this.join = true;
    },
    joinTeam() {
      var teamName = $("#joinTeamName").val();
      this.$ajax({
        method: "post",
        url: "/api/jointeam",
        data: {
          teamName: teamName,
          token: this.user.token
        }
      }).then(res => {
        if (res.data == "已加入小队!") {
          location.reload();
        } else {
           floatMessage(res.data)
           $(".trigger-info").click()
        }
        
      });
    }
  },
  mounted() {
    new Promise(resolve => {
      //获取用户信息
      this.$ajax({
        method: "get",
        url: "/api/getUserInfo"
      }).then(res => {
        this.$store.commit(types.USER, res.data);
        this.user = res.data;
        resolve(res.data);
      });
    }).then(data => {
      //获取问题
      this.$ajax({
        method: "post",
        url: "/api/getRandomQuestion?mobile=" + data.mobile
      }).then(res => {
        if (res.data.length == 0) {
          res.data = {name: "恭喜,你已答完所有题目!"}
        } 
        this.randomQuestion = res.data
      });
      //查询队伍
      this.$ajax({
        method: "get",
        url: "/api/teaminfo?mobile=" + data.mobile
      }).then(res => {
        this.teamInfo = res.data;
        if (res.data != undefined && res.data != "" && res.data != null) {
          this.haveTeam = true;
          var teamid = res.data.id;
          //获取队伍所有人
          this.$ajax({
            method: "get",
            url: "/api/teamall?teamid=" + teamid
          }).then(res => {
            this.teamall = res.data;
          });
        }
      });
    });
  },
  watch: {
    $route: function(to, from) {
    }
  },
  beforeCreate() {},
  created() {
    
  },
  components: {
    MemberItem
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
body {
  font: 14px "Lato", "Helvetica Neue", "Helvetica", sans-serif;
  min-height: 100%;
  background-color: #303133;
}
html {
  height: 100%;
}
.question {
  margin-top: 1%;
}
.question-simple {
  padding: 20px;
  background-color: #262729;
  height: 300px;
  width: 100%;
  color: #aaaaa9;
  font-size: 30px;
}

.question-des {
  padding: 20px;
  font-size: 30px;
  background-color: #3c3c3c;
  height: 300px;
  width: 100%;
  color: #aaaaa9;
}

.question-simple-q {
  height: 50%;
  width: 100%;
}
.question-des-body {
  height: 50%;
  width: 100%;
}
.question-simple-name {
  text-align: center;
  width: 100%;
  height: 80%;
  font-size: 40px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  margin-top: 30px;
}

.question-des-main {
  text-align: center;
  width: 100%;
  height: 80%;
  font-size: 22px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  margin-top: 30px;
}
.work-button {
  margin-top: 30px;
  text-align: center;
}
.tip {
  margin-top: 26px;
  color: #75757f;
}
.el-row {
  margin-bottom: 15px;
}
.team {
  height: 300px;
}
.team-main {
  width: 100%;
  background-color: #262729;
  color: #aaaaa9;
  font-size: 25px;
  text-align: center;
  padding: 10px;
}
.team-member {
  padding: 1px;
}
.team-member-info {
  color: #90bab5;
}
.team-msg {
  color: #f16967;
}
</style>
