<template>
  <div>
    <div class="work-top">
      <div class="main-left">
        <p
          style="font-size: 20px;font-weight: bold;color: #bccdd3"
        >QUESTION: {{randomQuestion.name}}</p>
        <hr style="border-top: 1px solid #3c3c3c;margin-top:10px;margin-bottom: 25px;">
        <div><textarea style="outline: none;" class="textarea-des" readonly v-text="randomQuestion.description"></textarea></div>
      </div>
      <div class="main-right">
        <textarea id="code" ref="code"></textarea>
      </div>
    </div>
    <div class="below">
      <div class="result">
        <template v-show="flag">
          <div ref="result">
            <p style="font-size: 20px;font-weight: bold;color: #bccdd3">RESULT:</p>
            <ol v-html="result" style="list-style-type: square"></ol>
          </div>
        </template>
        <div id="error" ref="error" v-show="!flag"></div>
      </div>
      <div class="task">
        <p style="font-size: 20px;font-weight: bold;color: #bccdd3">TASKS:</p>
        <ol ref="task">
          <li v-for="task in tasks" :key="task.id">
            {{task.taskQuestion}} = {{task.taskAnswer}}
            <hr style="border-top: 1px solid #3c3c3c;margin-top:10px;margin-bottom: 20px;">
          </li>
        </ol>
        <div class="submit">
          <div class="check">
            <button class="checkBtn"
              v-show="cheked"
              @click="cal()"
              style="background-color:#3c7dba;width:80px;border-radius:6px;border-color:#3c7dba;color:#303133"
            >检查</button>
            <button
              v-show="!cheked"
              @click="save()"
              style="background-color:rgba(0,0,0,0.2);width:80px;border-radius:6px;border-color:#3c7dba;color:#4467ab"
            >提交</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import NProgress from "NProgress";
export default {
  name: "Index",
  data() {
    return {
      result: "",
      consoleRes: "",
      editor: {},
      treeData: [],
      randomQuestion: {},
      tasks: {},
      flag: true,
      cheked: true,
      right: 0,
      submitPay: "",
      user: {},
      count: 1,
    };
  },
  methods: {
    init() {
      var that = this;
      this.editor = CodeMirror.fromTextArea(this.$refs.code, {
        lineNumbers: true,
        styleActiveLine: true,
        theme: "monokai",
        mode: "text/javascript",
        indentWithTabs: true,
        autofocus: true,
        matchBrackets: true,
        lineWrapping: true,
        completeSingle: false,
        // scrollbarStyle: null, //null隐藏滚动条
        extraKeys: {
          "Ctrl-Q": "autocomplete"
        } //
      });
      var editor = this.editor;
      editor.setSize("auto", "400px");
      // editor.setValue("//your code here");
      editor.setOption("extraKeys", {
        // Tab键换成4个空格
        Tab: function(cm) {
          var spaces = Array(cm.getOption("indentUnit") + 1).join("  ");
          cm.replaceSelection(spaces);
        },
        // F11键切换全屏
        F11: function(cm) {
          cm.setOption("fullScreen", !cm.getOption("fullScreen", true));
        },
        // Esc键退出全屏
        Esc: function(cm) {
          if (cm.getOption("fullScreen")) cm.setOption("fullScreen", false);
        }
      });
      editor.on("change", function() {
        that.codeChange();
      });
    },
    save() {
      var editor = this.editor;
      var answer = editor.getValue();
      this.$ajax({
        method: "post",
        url: "/api/saveAnswer",
        data: {
          questionId: this.randomQuestion.id,
          userMobile: this.user.mobile,
          answer: answer
        }
      })
        .then(res => {
          floatMessage(res.data);
          $(".trigger-info").click();
          if (res.status == 200) {
            this.$router.push(
              "/work/answerList?questionId=" + this.randomQuestion.id
            );
          }
        })
        .catch(err => {});
    },
    cal() {
      //初始化
      $('.checkBtn').attr("disabled", true)
      this.right = 0;
      this.count = 0
      this.flag = true;
      var task = this.randomQuestion.questionTasksList;
      var that = this;
      this.result = "";
      this.consoleRes = "";
      var editor = this.editor;
      var temp = "";
      var userAnswer = editor.getValue();
      var questionId = this.randomQuestion.id
       this.$nextTick(() => {
         this.getResult(questionId, userAnswer, task)
        })
    },
    log(str) {
      this.consoleRes += str + "\r\n";
    },
    codeChange() {
      this.cheked = true;
    },
    getResult(questionId, useranswer, task) {
      var that = this
      // var taskNum = parseInt(index) + parseInt(1)
                this.$ajax({
            method: "post",
            url: "/api/isAllRight",
            data: {
              questionId: questionId,
              useranswer: useranswer,
            },
            async: false, //或false,是否异步
          }).then(res => {
              var resultList = res.data
              for (var i in resultList) {
                var result =  resultList[i]
              if (result.wrong == false) {
                if (result.right == true) {
            //累计正确答案的数量
            this.right++;
            that.result +=
              "<li style='color:#67b04b;font-size: 22px'>正确!</li>";
            that.result +=
              "<hr style='border-top: 1px solid #3c3c3c;margin-top:10px;margin-bottom: 25px;'/>";
          } else {
            that.result +=
              "<li>Task: 我们希望得到的答案是:  " +
              task[i].taskAnswer +
              "</li>" +
              " <li style='color:#bb1b19'>您的答案是:  " +
              result.answer +
              "</li>";
            that.result +=
              "<hr style='border-top: 1px solid #3c3c3c;margin-top:10px;margin-bottom: 25px;'/>";
          }
              } else {
                //处理异常事件
                that.flag = false;
                this.treeData = [
                  {
                    text: "Catch Error(部分信息，可能因浏览器而异)",
                    nodes: [
                      {
                        // text: "Error message",
                        // nodes: [
                        //   {
                            text: result.answer,
                            color: "#bb1b19"
                        //   }
                        // ]
                      }
                    ]
                  }
                ];
                $("#error").treeview({ data: this.treeData,expanded: true });
               }
               if (i == task.length - 1 || result.wrong == true) {
                 $('.checkBtn').attr("disabled", false)
               }
              }


            var tasks = this.randomQuestion.questionTasksList;
            
            if (that.right == tasks.length) {
              that.cheked = false;
            }
            this.count = this.count + 1
})
    }
  },
  mounted() {
    var that = this;
    this.init();
    //获取随机问题
    var editor = this.editor;
    this.$ajax({
      method: "post",
      url: "/api/getQuestion/" + this.$route.query.id
    })
      .then(res => {
        this.randomQuestion = res.data;
        //设置题目
        editor.setValue(res.data.questionInit);
        //设置全局task
        that.tasks = res.data.questionTasksList;
      })
      .catch(err => {});
    console.log = this.log;
    this.user = JSON.parse(localStorage.user);
  },
  beforeCreate() {
    //替换console
    console.oldLog = console.log;
  },
  created() {},
  components: {}
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

body {
  font: 14px "Oxygen Mono", "Helvetica Neue", "Helvetica", sans-serif;
  min-height: 100%;
  display: flex;
  flex-direction: column;
  background-color: #303133;
}
html {
  height: 100%;
}
.CodeMirror {
  font: 20px "Oxygen Mono", Helvetica, Arial, sans-serif;
  /* overflow: auto; */
}

.main-left {
  letter-spacing: 0.1em;
  overflow: auto;
  flex-grow: 1;
  color: #aaaaa9;
  background-color: #1f2020;
  /* background-clip: content-box; */
  padding: 20px;
  width: 50%;
  height: 400px;
  margin: 10px;
}
.main-right {
  font-size: 20px;
  flex-grow: 1;
  width: 50%;
  height: 400px;
  margin: 10px;
}
.result {
  margin: 10px;
  height: 400px;
  width: 50%;
  padding: 10px;
  flex-grow: 1;
  overflow: auto;
}
.task {
  color: #aaaaa9;
  height: 200px;
  width: 50%;
  padding: 10px;
  flex-grow: 1;
}
.work-top {
  display: flex;
}
.below {
  display: flex;
}
#error {
  background-color: #3c3c3c;
  color: black;
}
.result {
  color: white;
  background-color: #1f2020;
}
.task {
  background-color: black;
  font-size: 20px;
  height: 400px;
  padding: 10px;
  margin: 10px;
  overflow: auto;
  display: flex;
  flex-direction: column;
}
.submit {
  margin-top: auto;
  /* push to bottom */
  align-self: flex-start;
  /* collapse to own width */
  margin-left: auto;
  /* centering */
  margin-right: auto;
}
.questionName {
  background-color: #77f93a;
}
.textarea-des {
  background-color: #1f2020;
  border: 0;
  height: 280px;
  width: 100%;
  overflow: auto;
  resize: none;
  font-size: 18px;
}

  ::-webkit-scrollbar {
  display: block !important;
  width: 0.5em !important;;
  overflow: auto !important;;
}
</style>
