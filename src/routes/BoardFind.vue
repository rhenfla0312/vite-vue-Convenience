<script>
import axios from 'axios'
export default {
  data() {
    return {
      id : this.$route.params.id,
      title : "",
      name : "",
      content : "",
      comments : [],
      comment_data : "",
      date : "",
      board_id : "",
      localName: localStorage.getItem("name"),
      comment_update_data: "",
    }
  },
  methods: {
    comment() {
      axios({
        method: 'POST',
        url : 'http://54.180.193.83:8081/boardcomment/',
        headers: {
          Authorization : `Bearer ${localStorage.getItem('access')}`
        },
        data : {
          comment : this.comment_data,
          username : localStorage.getItem('name'),
          board_id : this.board_id
        }
      }).then((res) => {
        console.log(res)
        this.$router.go()
      }).catch((error) => {
        console.log(error)
        alert("댓글을 입력해주세요")
      })
    },
    itemDelete(comment_id) {
      axios({
        method: 'DELETE',
        url : `http://54.180.193.83:8081/boardcomment/${comment_id}/`,
        headers: {
          Authorization : `Bearer ${localStorage.getItem('access')}`
        },
        data : {
          username : localStorage.getItem('name')
        }
      }).then((res) => {
        console.log(res)
        alert("삭제되었습니다")
        this.$router.go('')
      }).catch((error) => {
        console.log(error)
      })
    },
    deletes() {
      axios({
        method: 'delete',
        url : `http://54.180.193.83:8081/board/${this.board_id}/`,
        headers: {
          Authorization : `Bearer ${localStorage.getItem('access')}`
        },
        data : {
          username : this.name
        }
      }).then((res) => {
        console.log(res)
        alert("삭제되었습니다")
        this.$router.push('/board')
      }).catch((error) => {
        console.log(error)
      })
    },
    itemUpadate(comment_id) {
      axios({
        method : 'PUT',
        url : `http://54.180.193.83:8081/boardcomment/${comment_id}/`,
        headers: {
          Authorization : `Bearer ${localStorage.getItem('access')}`
        },
        data : {
          comment : this.comment_update_data,
          username : localStorage.getItem('name')
        }
      }).then((res) => {
        console.log(res)
        alert("수정이 완료되었습니다")
        this.$router.go()
      }).catch((error) => {
        console.log(error)
      })
    },
    itemUpdate__box(name) {
      if(this.localName == name) {
        this.update__box = !this.update__box
      }
    },
    update() {
      this.$router.push({
        name : 'BoardWrite',
        params : {
          id : this.board_id,
          title : this.title,
          content : this.content,
        }
      })
    },
    backBtn() {
      this.$router.push('/board')
    }
  },
  mounted() {
    axios({
      url : `http://54.180.193.83:8081/board/${this.id}/`,
      method : "GET"
    }).then((res) => {
      console.log(res)
      this.board_id = res.data.id
      this.title = res.data.title
      this.name = res.data.username
      this.content = res.data.content
      this.date = res.data.create_date
      this.comments = res.data.boardcomment
    }).catch((error) => {
      console.log(error)
    })
  }
}
</script>


<template>
  <div class="boardFind">
    <div class="boardFind__inner">
      <div class="boardFind__name">상세보기</div>
      <div class="boardFind__main">
        <div class="__dataBox">
          <div class="__title">{{ title }}</div>
          <div class="__infoBox">
            <div class="__name">{{ name }}</div>
            <div class="__date">{{ date.slice(0,-22) }}</div>
          </div>
        </div>
        <div class="__content">{{ content }}</div>
        <div class="__contentBtn">
          <div v-if="name == this.localName" @click="update()" class="__contentItem __contentUpdate">수정</div>
          <div v-if="name == this.localName" @click="deletes()" class="__contentItem __contentDelete">삭제</div>
          <div @click="backBtn()" class="__contentItem __contentBack">목록</div>
        </div>
        <!-- 댓글 -->
        <div class="comment">
          <div class="comment__title">전체댓글{{comments.length }}개</div>
          <div class="if" v-for="comment in comments" :key="comment">
            <div class="comment__info">
              <div class="__nickname">{{ comment.username }}</div>
              <div @click="itemUpdate__box(comment.nickname)" class="__comment">{{ comment.comment }}</div>
              <div class="__delete" v-if="comment.username == this.localName" @click="itemDelete(comment.id)"><i class="fa-solid fa-xmark"></i></div>
            </div>
            <!-- 댓글수정창 -->
            <div class="comment__update__box" v-if="comment.nickname == this.localName" :class="{ update__box : update__box }">
              <div class="comment__update__text">
                <!-- :value에 원래 댓글 입력하니 오류가 걸려 일단 제거 -->
                <textarea class="__text"></textarea>
              </div>
              <div @click="itemUpadate(comment.id)" class="comment__update__btn">
                <button class="__btn">수정</button>
              </div>
            </div>
          </div>
          <div class="comment__else__info" v-show="this.comments.length == 0">등록된 댓글이 없습니다</div>
          <!-- 페이지네이션 -->
          <div class="comment__page">
            <i class="fa-solid fa-1 first"></i>
            <i class="fa-solid fa-2"></i>
            <i class="fa-solid fa-3"></i>
          </div>
          <div class="comment__box">
            <div class="comment__text">
              <textarea v-model="comment_data" class="__text"></textarea>
            </div>
            <div @click="comment" class="comment__btn">
              <button class="__btn">등록</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  a {
    text-decoration: none;
    color: #333;
  }
  .boardFind {
    position: relative;
    top: 125px;
    .boardFind__inner {
      width: 1200px;
      margin: auto;
      position: relative;
      .boardFind__name {
        font-size: 25px;
        font-weight: bold;
        line-height: 3;
        color : #29367c;
        border-bottom: 2px solid #3b4890;
      }

      // 메인
      .boardFind__main {
        margin-top: 10px;
        font-size: 20px;
        .__dataBox {
          margin: 16px 0 29px;
          padding-bottom: 11px;
          border-bottom: 1px solid #333;
          .__infoBox {
            display: flex;
            font-size: 17px;
            color: #333;
            .__name {
              &::after {
                content: "";
                height: 15px;
                width: 1px;
                background: #ccc;
                margin-left: 10px;
                display: inline-block;
                margin-right: 10px;
              }
            }
          }
        }
        .__content {
          height: 100%;
          min-height: 500px;
          margin-top: 50px;
        }
        .__contentBtn {
          display: flex;
          justify-content: end;
          padding-bottom: 100px;
          padding-top: 5px;
          border-top: 2px solid #3b4890;
          text-align: center;
          .__contentItem {
            outline: none;
            width: 100px;
            background: #3b4890;
            border-color: #29367c;
            color: #fff;
            cursor: pointer;
            &:hover {
              opacity: .8;
            }
          }
          .__contentDelete {
            margin-left: 10px;
            margin-right: 10px;
          }
        }
        .comment {
          min-height: 400px;
          font-size: 18px;
          .comment__title {
            border-bottom: 2px solid #3b4890;
            text-align: start;
          }
          .if {
            text-align: start;
            border-bottom: 1px solid #333;
            &:last-child {
              border-bottom: none
            }
            .comment__info {
              display: flex;
              padding: 10px 0;
              position: relative;
              padding: 10px 0 10px 0;
              .__comment {
                position: absolute;
                cursor: pointer;
                left: 400px;
              }
              .__delete {
                cursor: pointer;
                position: absolute;
                right: 3px;
              }
            }
            .comment__update__box {
              display: none;
            }
            .comment__update__box.update__box {
              display: block;
              border-top: 2px solid #3b4890;
              padding: 10px 0 10px 0;
              .comment__update__text {
                .__text {
                  width: 90%;
                  height: 100px;
                  outline: none;
                  resize: none;
                  margin-left: 55px;
                  font-size: 18px;
                }
              }
              .comment__update__btn {
                text-align: end;
                margin-right: 66px;
                .__btn {
                  outline: none;
                  width: 100px;
                  background: #3b4890;
                  border-color: #29367c;
                  color: #fff;
                  &:hover {
                    opacity: .8;
                  }
                }
              }
            }
          }
          .comment__else__info {
            border-bottom: 1px solid #333;
            padding-top: 10px;
            padding-bottom: 9px;
          }
          .comment__page {
            margin: auto;
            padding: 10px 0 10px 0;
            text-align: center;
            i {
              margin-right: 10px;
              cursor: pointer;
            }
            .first {
              color : red;
            }
          }
          .comment__box {
            border-top: 2px solid #3b4890;
            padding: 10px 0 10px 0;
            .comment__text {
              .__text {
                width: 100%;
                height: 100px;
                outline: none;
                resize: none;
                font-size: 18px;
              }
            }
            .comment__btn {
              text-align: end;
              .__btn {
                outline: none;
                width: 100px;
                background: #3b4890;
                border-color: #29367c;
                color: #fff;
                &:hover {
                  opacity: .8;
                }
              }
            }
          }
        }
      }
    }
  }
</style>