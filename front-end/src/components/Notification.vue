<template>
   <div class="container-fluid photos">
      <div class="row justify-content-center">

        <div class="col-md-8 pt-4">
            <div class="mb-5" data-aos="fade-up">
              <h2 class="mb-5 text-center text-light">읽지 않은 알림</h2>
              <div class="text-right"><button class="btn btn-outline-success" @click="check='scrap'">모두 읽음</button></div>
              <ul class="nav nav-tabs" id="myTab" role="tablist">
                <li class="nav-item">
                  <a class="nav-link" :class="{'active': check=='all'}" @click="check='all'">전체 <span class="badge badge-primary">{{allNoti.length}}</span></a>
                </li>
                <li class="nav-item">
                  <a class="nav-link" :class="{'active': check=='follow'}" @click="check='follow'">팔로우 <span class="badge badge-dark">{{followNoti.length}}</span></a>
                </li>
                <li class="nav-item">
                  <a class="nav-link" :class="{'active': check=='like'}" @click="check='like'">좋아요 <span class="badge badge-dark">{{likeNoti.length}}</span></a>
                </li>
                <li class="nav-item">
                  <a class="nav-link" :class="{'active': check=='scrap'}" @click="check='scrap'">스크랩 <span class="badge badge-dark">{{scrapNoti.length}}</span></a>
                </li>
              </ul>
            
              <table class="table text-light">
                <tbody v-if="check=='all'">
                  <tr v-for="(item, index) in allNoti" :key="`follow${index}`">
                    <td v-if="item.category == 'follow'">
                      <router-link :to="'/mypage/'+item.user_id"><span style="font-weight: 550;">{{item.user_id}} </span></router-link>님이 회원님을 팔로우하기 시작했습니다.
                      <small>{{require('moment')(item.timestamp, "YYYY-MM-DD HH:mm:ss").fromNow()}}</small>
                    </td>
                    <td v-if="item.category == 'like'">
                      <router-link :to="'/mypage/'+item.user_id"><span style="font-weight: 550;">{{item.user_id}} </span></router-link>님이 회원님의 게시물을 좋아합니다.
                      <small>{{require('moment')(item.timestamp, "YYYY-MM-DD HH:mm:ss").fromNow()}}</small>
                    </td>
                    <td v-if="item.category == 'scrap'">
                      <router-link :to="'/mypage/'+item.user_id"><span style="font-weight: 550;">{{item.user_id}} </span></router-link>님이 회원님의 게시물을 스크랩했습니다.
                      <small>{{require('moment')(item.timestamp, "YYYY-MM-DD HH:mm:ss").fromNow()}}</small>
                    </td>
                  </tr>
                </tbody>
                <tbody v-if="check=='follow'">
                  <tr v-for="(item, index) in followNoti" :key="`follow${index}`">
                    <td>
                      <router-link :to="'/mypage/'+item.user_id"><span style="font-weight: 550;">{{item.user_id}} </span></router-link>님이 회원님을 팔로우하기 시작했습니다.
                      <small>{{require('moment')(item.timestamp, "YYYY-MM-DD HH:mm:ss").fromNow()}}</small>
                    </td>
                  </tr>
                </tbody>
                <tbody v-if="check=='like'">
                  <tr v-for="(item, index) in likeNoti" :key="`follow${index}`">
                    <td>
                      <router-link :to="'/mypage/'+item.user_id"><span style="font-weight: 550;">{{item.user_id}} </span></router-link>님이 회원님의 게시물을 좋아합니다.
                      <small>{{require('moment')(item.timestamp, "YYYY-MM-DD HH:mm:ss").fromNow()}}</small>
                    </td>
                  </tr>
                </tbody>
                <tbody v-if="check=='scrap'">
                  <tr v-for="(item, index) in scrapNoti" :key="`follow${index}`">
                    <td>
                      <router-link :to="'/mypage/'+item.user_id"><span style="font-weight: 550;">{{item.user_id}} </span></router-link>님이 회원님의 게시물을 스크랩했습니다.
                      <small>{{require('moment')(item.timestamp, "YYYY-MM-DD HH:mm:ss").fromNow()}}</small>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
        </div>
      </div>
    </div>

</template>

<script>
import http from '../http-common.js';
require('moment/locale/ko');

export default {
  data() {
    return {
      allNoti: [],
      followNoti: [],
      likeNoti: [],
      scrapNoti: [],
      check: 'all',
    }
  },
  methods: {
    fetchNoti() {
       http
        .get(`/notification/uncheckedList/${this.$store.state.user_id}`)
        .then(response => {
            this.allNoti = response.data.resvalue;
            this.allNoti.reverse().slice(0,10);
            console.log(this.allNoti)
        })
        .catch(e => console.log(e))
    },
    fetchCategoryNoti(category) {
      http
        .get(`/notification/categoryUncheckedList/${this.$store.state.user_id}/${category}`)
        .then(response => {
            if (category == 'follow') {
              this.followNoti = response.data.resvalue;
              this.followNoti.reverse().slice(0,10);
            }else if (category == 'like') {
              this.likeNoti = response.data.resvalue;
              this.likeNoti.reverse().slice(0,10);
            }else {
              this.scrapNoti = response.data.resvalue;
              this.scrapNoti.reverse().slice(0,10);
            }
        })
        .catch(e => console.log(e))
    },
  },
  mounted() {
    this.fetchNoti();
    this.fetchCategoryNoti('follow');
    this.fetchCategoryNoti('like');
    this.fetchCategoryNoti('scrap');
  }
}
</script>

<style scoped>
small {
  color: gray;
}
</style>