<template>
  <div id="rental" class="container globalBoard">
    <div class="body_container">
      <custom-navigation1 v-if="navigationKind === 1"></custom-navigation1>
      <custom-navigation2 v-else-if="navigationKind === 2"></custom-navigation2>
      <custom-navigation3 v-else-if="navigationKind === 3"></custom-navigation3>
      <div id="notice_contents">
        <div id="content_name">
          <div>{{content_name}}</div>
        </div>
        <div id="content_body">
          <div>
            <div id="e_content">
              <p>제목</p>
              <div id="div_content">
                <textarea name="e_textarea_title" id="ip_box" v-model="title" rows="1"></textarea>
              </div>
            </div>
            <div id="e_content">
              <p>내용</p>
              <div id="div_content">
                <textarea name="e_textarea_content" id="ip_box" v-model="content" rows="15"></textarea>
              </div>
            </div>
            <div id="e_content">
              <p>첨부파일</p>
              <div id="div_content">
                <div id="div_file">
                  <input
                    id="ip_box"
                    type="file"
                    ref="file"
                    @change="handleFileUpload($event.target.name, $event.target.files)"
                    @drop="handleFileUpload($event.target.name, $event.target.files)"
                  />
                  <p>첨부파일을 마우스로 끌어 놓으세요</p>
                </div>
                <div id="div_options">
                  <div>
                    <button id="bt_submit" type="button" @click="submit()">게시하기</button>
                  </div>
                  <div id="notice_hide">
                    <div id="div_notice">
                      <input type="checkbox" v-model="notice" />&nbsp 공지 등록
                    </div>
                    <div v-if="invisibleMode" id="div_hide">
                      <input id="check_hide" type="checkbox" v-model="hide" />&nbsp 비밀글 등록
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
#div_options {
  width: 90%;
  display: flex;
}
textarea {
  resize: none;
}
#e_content > p {
  width: 100px;
  text-align: center;
}
#e_content {
  display: flex;
}
#div_content {
  width: 100%;
}
#div_file {
  border: 1px solid gray;
  width: 90%;
  padding: 2px;
  margin-top: 16px;
}
#ip_box {
  margin-top: 16px;
  width: 90%;
}
#bt_submit {
  width: 64px;
  height: 36px;
  border-radius: 10px;
  background-color: #003e8f;
  font-size: 13px;
  font-weight: 500;
  text-align: center;
  color: #ffffff;
  margin-top: 10px;
}

#notice_hide {
  display: flex;
  margin-left: auto;
}

#div_notice,
#div_hide {
  margin: 15px 15px;
}
</style>


<script>
import customNavigation1 from "@/pages/2nd_menu/custom_navigation";
import customNavigation2 from "@/pages/3rd_menu/custom_navigation";
import customNavigation3 from "@/pages/4th_menu/custom_navigation";
import router from "@/router";
import axios from "axios";
import { global } from "@/global";
import { mapState } from "vuex";

export default {
  name: "enroll",

  components: {
    customNavigation1,
    customNavigation2,
    customNavigation3
  },

  computed: {
    ...mapState(["fail_access"]),
    ...mapState(["check_admin"])
  },

  created() {
    this.fail_access(!this.$session.exists());
  },

  data: () => ({
    invisibleMode: false,
    title: "",
    content: "",
    file: [],
    file_name: "",
    boardKind: 0,
    boardId: 0,
    navigationKind: 0,
    content_name: "물품대여",
    modify: false,
    notice: false,
    hide: false
  }),

  mounted() {
    this.boardKind = this.$route.query.boardKind;
    this.boardKindFilter(this.boardKind);

    if (this.$route.query.update == 1) {
      this.modify = true;
      this.title = this.$route.params.pre_title;
      this.content = this.$route.params.pre_content;
      this.boardKind = this.$route.params.pre_boardKind;
      this.boardId = this.$route.params.pre_boardId;
    }
    this.allow_admin();
    console.log(this.$session.get("member_id"));
  },

  watch: {
    file_name: function() {
      this.file_name = this.file_name;
    },
    modify: function() {
      return this.modify;
    }
  },

  methods: {
    allow_admin() {
      if (this.boardKind == 3 || this.boardKind == 4 || this.boardKind == 5) {
        if (this.check_admin()) {
        } else {
          alert("관리자만 작성가능한 페이지입니다.");
          this.$router.go(-1);
        }
      }
    },
    boardKindFilter(kind) {
      var self = this;
      if (kind == 1) {
        self.content_name = "학생청원";
        self.navigationKind = 1;
      } else if (kind == 2) {
        self.content_name = "게시판";
        self.navigationKind = 1;
      } else if (kind == 3) {
        self.content_name = "회의보고";
        self.navigationKind = 2;
      } else if (kind == 4) {
        self.content_name = "재정보고";
        self.navigationKind = 2;
      } else if (kind == 5) {
        self.content_name = "서식자료실";
        self.navigationKind = 2;
      } else if (kind == 6) {
        self.content_name = "물품대여";
        self.navigationKind = 3;
        self.invisibleMode = true;
      } else if (kind == 7) {
        self.content_name = "통학버스 및 귀향버스";
        self.navigationKind = 3;
      }
    },
    handleFileUpload(name, files) {
      var self = this;
      self.file = files[0];
      self.file_name = files[0].name;
    },
    submit() {
      var self = this;

      if (self.title.length < 1) {
        alert("제목을 입력해주세요");
      } else if (self.content < 1) {
        alert("내용을 입력해주세요");
      } else {
        if (self.modify) {
          self.update();
        } else {
          self.createBoard();
        }
      }
    },
    update() {
      var self = this;
      let config = {
        headers: {
          "x-access-token": self.$session.get("member_token")
        }
      };
      const formData = new FormData();
      formData.append("boardKind", self.boardKind);
      formData.append("title", self.title);
      formData.append("content", self.content);
      formData.append("userFile", self.file);
      formData.append("boardId", self.boardId);
      formData.append("notice", self.notice);
      formData.append("boardSecret", self.visibleMode);

      axios
        .post(`${global.base}/board/update`, formData, config)
        .then(response => {
          if (response.data.ans == "success") {
            console.log("게시판 수정");
            alert("정상적으로 수정되었습니다.");
          }
        })
        .catch(error => {
          console.error(
            error.response + "에러 발생, 게시판 리스트를 불러올 수 없음"
          );
        });

      self.$router.push({
        name: "detail",
        query: {
          boardKind: self.boardKind,
          boardId: self.boardId
        }
      });
    },
    createBoard() {
      var self = this;
      let config = {
        headers: {
          "x-access-token": self.$session.get("member_token"),
          "Content-Type": "multipart/form-data"
        }
      };

      const formData = new FormData();
      formData.append("boardKind", self.boardKind);
      formData.append("title", self.title);
      formData.append("content", self.content);
      formData.append("userFile", self.file);
      formData.append("notice", self.notice);
      formData.append("boardSecret", self.hide);
      axios
        .post(`${global.base}/board/create`, formData, config)
        .then(response => {
          if (response.data.ans == "success") {
            console.log("게시판 생성");
            alert("정상적으로 등록되었습니다.");
          }
        })
        .catch(error => {
          console.error(error.response + "에러 발생, 업로드 오류");
        });
      self.getContentsList();
    },
    getContentsList() {
      var self = this;

      axios
        .post(`${global.base}/board/all`, { boardKind: self.boardKind })
        .then(response => {
          var rentalData = response.data[0];
          console.log(rentalData[0]);
          for (var item in rentalData) {
            if (rentalData[item].author == self.$session.get("member_id")) {
              self.boardId = rentalData[item].boardId;
              self.getDetail();
              break;
            }
          }
        })
        .catch(error => {
          console.error(
            error.response + "에러 발생, 게시판 리스트를 불러올 수 없음"
          );
        });
    },
    getDetail() {
      var self = this;
      self.$router.push({
        name: "detail",
        query: {
          boardKind: self.boardKind,
          boardId: self.boardId
        }
      });
    }
  }
};
</script>
