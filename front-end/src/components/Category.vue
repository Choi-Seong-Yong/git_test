<template>
    <div class="container-fluid photos">
      <div class="row align-items-stretch">
        <!-- <div @mouseover="showPagination" @mouseleave="hiddenPagination" class="col-6 col-md-6 col-lg-4" data-aos="fade-up" v-for="item in Items" :key="item.id"> -->
        <div class="col-6 col-md-6 col-lg-4" data-aos="fade-up" v-for="item in Items" :key="item.id">
          <div class="d-block photo-item">
            <div class="all-scroll pos-relative mt-50">
              <div class="swiper-scrollbar"></div>
              <div class="swiper-container oflow-visible" data-slide-effect="coverflow" data-autoheight="false"  data-swiper-wheel-control="true"
                                        data-swiper-speed="1000" data-swiper-margin="25" data-swiper-slides-per-view="1"
                                        data-swiper-breakpoints="true" data-swiper-autoplay="false" data-scrollbar="true"
                                        data-swiper-loop="false" data-swpr-responsive="[1, 2, 1, 2]">
                <div class="swiper-wrapper">
                    <div class="swiper-slide" v-for="img in item.imageList" :key="img.index">
                        <div v-on:click="goDetail(item.content_id)" :class="img.filter" style="width:100%; height:300px">
                          <img :src="img.image_url" alt="Image"/>
                        </div>
                    </div>
                </div>
                <div class="swiper-pagination"  slot="pagination"></div>
                <div class="swiper-button-prev" slot="button-prev"></div>
                <div class="swiper-button-next" slot="button-next"></div>
              </div>
            </div>
            <!-- <img :src="item.image.image_url" alt="Image" class="img-fluid"/> -->
              <div class="photo-text-more">
                <!-- <div class="heading mx-2 ellipsis" v-html="content_val"></div> -->
                <h3 class="heading mx-2 ellipsis">{{item.content_id}} {{item.content_val}}</h3>
                <span class="meta">
                  <div class="mb-3 my-3 d-flex justify-content-around size">
                    <div @click="clickHeart(item.content_id)">
                      <i class="icon-heart" v-if="item.user_like"></i>
                      <i class="icon-heart-o" v-else></i>
                    </div>
                    <div v-on:click="clickFollow()">
                      <i class="icon-check" v-if="follow"></i>
                      <i class="icon-user-plus" v-else></i>
                    </div>
                    <div v-on:click="clickBell()">
                      <i class="icon-bell" v-if="bell"></i>
                      <i class="icon-bell-o" v-else></i>
                    </div>
                  </div>
                </span>
              </div>
          </div>
        </div>
      </div>
      <div class="text-white text-center" v-if="this.contentErrorMsg">
        <h5>{{this.contentErrorMsg}}</h5>
      </div>
    </div>
</template>

<script>
// import axios from 'axios'
import $ from "jquery"
import http from '../http-common';
import store from '../store'
export default {
  data() {
    return {
      follow: false,
      bell: false,
      errored: false,
      uid: "",
      Items: [],
      contentIds: [],
      contentErrorMsg: "",
    }
  },
  methods: {
    getLike() {
      this.uid = store.state.user_id
      http
        .get('/userLike/userLikeList/' + this.uid)
        .then((res) => {
          for (var i = 0; i < res.data.resvalue.length; i++) {
            this.contentIds.push({
              con_id: res.data.resvalue[i].content_id
            })
          }
        })
        .catch(()=>{
          this.errored = true;
        })
    },
    getData() {
      http
        .get('/content/contentMyList/' + this.uid)
        .then((res)=>{
          if (res.data.resValue.length > 0) {
            this.contentErrorMsg = ""
            if (res.data.resmsg == "타임라인 출력 성공") {
              for (var i = 0; i < res.data.resValue.length; i++) {
                for (var j = 0; j < this.contentIds.length; j++) {
                  if (res.data.resValue[i].content_id == this.contentIds[j].con_id) {
                    res.data.resValue[i].user_like = true
                  }
                }
                this.Items.push({
                  content_id: res.data.resValue[i].imageList[0].content_id,
                  image: {
                    image_url: res.data.resValue[i].imageList[0].image_url,
                    filter: res.data.resValue[i].imageList[0].filter
                  },
                  imageList: res.data.resValue[i].imageList,
                  content_val: res.data.resValue[i].content_val,
                  timestamp: res.data.resValue[i].timestamp,
                  user_like: res.data.resValue[i].user_like,
                })
              }
            }
          } else {
            this.contentErrorMsg = "게시물이 없습니다."
          }
        })
        .catch(()=>{
          this.errored = true;
        })
    },
    goDetail: function(con_id) {
      this.$router.push({
        name: 'bio',
        params: {
          cid: con_id
        }
      })
    },
    clickHeart(num) {
      var idx = 0
      for (idx = 0; idx < this.Items.length; idx++) {
        if (this.Items[idx].content_id == num) {
          if (this.Items[idx].user_like == false) {
            this.Items[idx].user_like = true
            http
              .post('/userLike/like', {
                content_id: this.Items[idx].content_id,
                timestamp: this.Items[idx].timestamp,
                user_id: this.uid
              })
              .catch(()=>{
                this.errored = true;
              })
          } else {
            this.Items[idx].user_like = false       
            http
              .delete('/userLike/dislike', {
                data: {
                  content_id: this.Items[idx].content_id,
                  timestamp: this.Items[idx].timestamp,
                  user_id: this.uid
                }
              })
              .catch(()=>{
                this.errored = true;
              })
          }
        }
      }
    },
    clickFollow() {
      if (this.follow === true) {
        this.follow = false;
      } else {
        this.follow = true;
      }
    },
    clickBell() {
      if (this.bell === true) {
        this.bell = false;
      } else {
        this.bell = true;
        this.$store.commit('setModalText', '신고가 접수되었습니다.');
        document.getElementById('modalBtn').click();
      }
    }
  },
  created() {
    this.getLike()
    this.getData()
  },
  mounted() {
    $('html').scrollTop(0);
  },
  updated(){
    let recaptchaScripta = document.createElement('script')
    recaptchaScripta.setAttribute('type',"text/javascript")
    recaptchaScripta.setAttribute('src', "./theme/js/script.js")
    document.body.appendChild(recaptchaScripta)
    let recaptchaScriptb = document.createElement('script')
    recaptchaScriptb.setAttribute('type',"text/javascript")
    recaptchaScriptb.setAttribute('src', "./theme/js/swiper.js")
    document.body.appendChild(recaptchaScriptb)
  }
}
</script>

<style>
  #textarea {
    width: 100%;
  }
  #bg {
    background-color: rgba(255, 255, 255, 0.2);
  }
  #text-color {
    color: white;
  }
  .ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    word-wrap: break-word;
    line-height: 2;
    height: 6rem;
  }
  .size {
    font-size: 2em;
  }
</style>