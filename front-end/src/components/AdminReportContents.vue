<template>
    <div class="container-fluid photos">
        <div class="row align-items-stretch">
            <!-- 게시물 하나 -->
            <div class="col-6 col-md-6 col-lg-4" data-aos="fade-up" style="padding: 10px 10px" v-for="con in contents"
                :key="con.contentId">
                <div class="d-block photo-item">
                    <!-- 이미지 처리 -->
                    <div class="polaroid" v-if="readContents.includes(con.contentId)">
                        <div :class="con.images[0].filter" style="width:100%; height:100%">
                            <img :src="con.images[0].imageUrl" alt="Image" class="img-fluid pa m-0"
                                style="box-shadow: 3px 3px 3px;" />
                        </div>
                    </div>
                    <div class="polaroid" v-show="con.dislike >= 5 && !readContents.includes(con.contentId) ">
                        <div style="width:100%; height:100%">
                            <img :src="con.images[0].imageUrl" alt="Image" class="img-fluid pa blur m-0"
                                style="box-shadow: 3px 3px 3px;" />
                            <p class="centertext text-white">신고횟수 : {{con.dislike}}</p>
                            <button class="centerbutton btn btn-primary btn-sm"
                                @click="readReCon(con.contentId)">보기</button>
                        </div>
                    </div>
                    <!-- 마우스 오버 했을 때 -->
                    <div class="photo-text-more" v-if="con.dislike < 5 && readContents.includes(con.contentId)">
                        <div class="">
                            <div class="d-block photo-item">
                                <div class="postcard">
                                    <div class="content">
                                        <!-- 누르면 상세 페이지로 -->
                                        <div v-on:click="goDetail(con.contentId)">
                                            <!-- 우표 -->
                                            <div class="stamp-cover">
                                                <div class="stamp"
                                                    style=" margin:1px; float:right; background-color:white; height:50px; width:50px;">
                                                </div>
                                            </div>
                                            <div class="stamp-img"
                                                style="top:25px;right:25px;height:43px;width:43px;background-color:white;">
                                            </div>
                                            <div style="width:37px;height:37px" class="stamp-img"
                                                :class="con.profileFilter">
                                                <img :src="con.profileUrl"
                                                    style="width:37px;height:37px; background: none;" />
                                            </div>
                                            <img src="../../public/theme/images/stamp1.png"
                                                style="width:45px;height:45px;" alt="Postage mark" class="postmark">
                                            <!-- 우편 내용 -->
                                            <div class="mail-title offset-1 col-9 mt-2 ml-3 pb-"
                                                style="text-align:left;">
                                                <p class="mail-title-val">Dear {{ loginId }}</p>
                                            </div>
                                            <div class="mail-message offset-2 pt-0 pb-0 col-8 ellipsis mail-message-val"
                                                v-html="con.contentValue">{{ con.contentValue }}</div>
                                            <div class="col-11 col-offset-1 pt-0 pr-0 mail-from-val">from
                                                {{ con.userId }}</div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
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
    import $ from "jquery"
    import http from "../http-common"
    import store from '../store'
    export default {
        data() {
            return {
                temp_contents: [],
                contents: [{
                    contentId: "",
                    contentValue: "",
                    timestamp: "",
                    userId: "",
                    imageLength: 0,
                    dislike: 0,
                    images: [{
                        imageUrl: "",
                        filter: "",
                    }],
                    profileUrl: "",
                    profileFilter: "",
                    report_category: [],
                }],
                loginId: store.state.user_id,
                contentErrorMsg: "",
                errored: false,
                readContents: [],
            }
        },
        methods: {
            readReCon(cid) {
                this.readContents.push(cid)
            },
            getData() {
                if (this.contents[0].contentId == "") {
                    this.contents = []
                }
                http
                    .get('/userReport/adminreportContentList')
                    .then((res) => {
                        if (res.data.resValue.length > 0) {
                            this.contentErrorMsg = ""
                            for (var idx = 0; idx < res.data.resValue.length; idx++) {
                                this.contents.push({
                                    contentId: res.data.resValue[idx].content_id,
                                    contentValue: res.data.resValue[idx].content_val.replace(/\n/g,
                                        "<br />"),
                                    timestamp: res.data.resValue[idx].timestamp,
                                    userId: res.data.resValue[idx].user_id,
                                    imageLength: res.data.resValue[idx].imageLength,
                                    images: [{
                                        imageUrl: res.data.resValue[idx].imageList[0].image_url,
                                        filter: res.data.resValue[idx].imageList[0].filter,
                                    }],
                                    dislike: res.data.resValue[idx].dislike,
                                    profileUrl: res.data.resValue[idx].profile_url,
                                    profileFilter: res.data.resValue[idx].profile_filter,
                                })
                                http
                                    .get('/userReport/getreportcategory/' + res.data.resValue[idx].content_id)
                                    .then((resault) => {
                                        this.contents.push({
                                            report_category: resault.data.resValue,
                                        })

                                    })
                                    .catch(() => {
                                        this.errored = true;
                                    })

                            }
                        } else {
                            this.contentErrorMsg = "타임라인이 없습니다."
                        }
                    })
                    .catch(() => {
                        this.errored = true;
                    })
            },
            goDetail: function (con_id) {
                this.$router.push({
                    name: 'bio',
                    params: {
                        cid: con_id
                    }
                })
            },
        },
        mounted() {
            this.getData()
            $('html').scrollTop(0);
            this.$nextTick(() => {
                // 모든 화면이 렌더링된 후 호출됩니다.
                // console.log(document.querySelectorAll("ul").length)
                $('.js-clone-nav').each(function () {
                    var $this = $(this);
                    $this.clone().attr('class', 'site-nav-wrap').appendTo('.site-mobile-menu-body');
                });
                document.querySelector("ul").remove();
                // document.querySelector("ul").remove();
            });
        },
    }
</script>
<style scoped>
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

    .pa {
        position: relative;
    }

    .centertext {
        position: absolute;
        text-align: center;
        padding: 5px;
        display: block;
        top: 37%;
        left: 50%;
        background-color: #333;
        font-size: 13px;
        transform: translateY(-50%);
        transform: translateX(-50%);
    }

    .centerbutton {
        position: absolute;
        text-align: center;
        display: block;
        top: 45%;
        left: 50%;
        margin-top: 70px;
        background-color: #333;
        transform: translateY(-50%);
        transform: translateX(-50%);
    }

    .blur {
        -webkit-filter: blur(5px);
        -moz-filter: blur(5px);
        -o-filter: blur(5px);
        -ms-filter: blur(5px);
        filter: blur(5px);
        opacity: 0.7;
        /* transform: scale(1); */
        overflow: hidden;
        display: block;
        width: 80%;
        height: 80%;
    }

    .modal-dialog {
        width: 30%;
        height: 50%;
    }

    .modal-content {
        height: auto;
        min-height: 100%;
    }

    .mydiv {
        margin-top: 7em;
    }

    .size {
        font-size: 1em;
    }
</style>