<template>
    <div class="container-fluid photos">
        <div class="row justify-content-center">

            <div class="col-md-8 pt-4">
                <!-- 제목 -->
                <div class="row mb-5" data-aos="fade-up">
                    <div class="col-12">
                        <h2 class="text-white mb-4 text-center">친구 찾기</h2>
                    </div>
                </div>
                
                <!-- 친구 검색 바 -->
                <div class="d-flex justify-content-center" v-if="IdButton">
                    <input type="search" v-model="inputIdData" class="col-7 btn btn-white text-danger" placeholder="아이디를 입력해주세요">
                </div>
                <div class="" v-else>
                    <div class="col-12 d-flex justify-content-center">
                        <input type="search" class="col-7 btn btn-white text-danger" @keyup.enter.prevent='searchInput' @keydown.space.prevent='searchInput' id="itrlone" v-model="inputInterestData" placeholder="관심사를 입력해주세요">
                        <div class="btn-toolbar" role="toolbar" aria-label="Toolbar with button groups">
                            <div class="btn-group mx-1" role="group" aria-label="First group">
                                <button type="submit" class="btn btn-primary" @click="getInterests()">
                                    <i class="icon-search"></i>
                                </button>
                                <button type="button" class="btn btn-danger" @click="resetData()">
                                    <i class="icon-refresh"></i>
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="mt-5 text-white col-12 d-flex justify-content-center">
                        <p v-if="search">관심사 입력 후 스페이스바 혹은 엔터 키를 눌러주세요</p>
                        <div class="btn-toolbar" role="toolbar" aria-label="Toolbar with button groups">
                            <div v-for="data in searchData" :key="data.id" class="btn-group mx-1" role="group" aria-label="First group">
                                <button type="button" class="btn btn-info">{{ data }}</button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- 친구 찾기 목록 -->
                <div class="mt-5 row justify-content-center">
                    <div @click="IdClick()" class="d-block col-6">
                        <button v-show="IdButton" type="button" class="btn btn-info col-12 btn-lg">ID</button> 
                        <button v-show="!IdButton" type="button" class="btn btn-dark col-12 btn-lg">ID</button>
                    </div>
                    <div @click="InterestClick()" class="d-block col-6">
                        <button v-show="!InterestButton" type="button" class="btn btn-dark col-12 btn-lg">INTEREST</button>
                        <button v-show="InterestButton" type="button" class="btn btn-info col-12 btn-lg">INTEREST</button>
                    </div>

                    <div>
                        <div v-show="IdButton" v-for="result in resultIds" :key="result.id" class="my-4">
                            <div v-if="result.user_id" class="media position-relative">
                                <img v-if="result.profile_url" :src="result.profile_url" class="rounded-circle mb-3" width="130px" height="130px" style="object-fit: cover;">
                                <img v-else src="https://t1.daumcdn.net/qna/image/1542632018000000528" class="mb-3" width="130px" height="130px" style="object-fit: cover;">
                                <div class="media-body">
                                    <div class="notification align-self-center ml-3">
                                        <h4 class="mt-2 text-white">{{ result.user_id }} 님</h4>
                                        <p v-if="result.description">{{ result.description }}</p>
                                        <p v-else> 반갑습니다 </p>
                                        <router-link :to="'/mypage/'+ result.user_id" class="text-primary">Go {{result.user_id}} page</router-link>
                                    </div>
                                </div> 
                            </div>
                        </div>
                        <div v-show="IdButton">
                            <h3 class="m-3 text-white">{{idErrorMsg}}</h3>
                        </div>

                        <div v-show="InterestButton" v-for="result in resultInterests" :key="result.id" class="my-4">
                            <div v-if="result.user_id" class="media position-relative">
                                <img v-if="result.profile_url" :src="result.profile_url" class="rounded-circle mb-3" width="130px" height="130px" style="object-fit: cover;">
                                <img v-else src="https://t1.daumcdn.net/qna/image/1542632018000000528" class="mb-3" width="130px" height="130px" style="object-fit: cover;">
                                <div class="media-body">
                                    <div class="notification align-self-center ml-3">
                                        <h4 class="mt-2 text-white">{{ result.user_id }} 님</h4>
                                        <p v-if="result.interest">관심사: {{ result.interest }}</p>
                                        <!-- <p v-else> 관심사: 없음</p> -->
                                        <router-link :to="'/mypage/'+ result.user_id" class="text-primary">Go {{result.user_id}} page</router-link>
                                    </div>
                                </div> 
                            </div>
                        </div>
                        <div v-show="InterestButton">
                            <h3 class="m-3 text-white">{{interestErrorMsg}}</h3>
                        </div>
                    </div>
                </div>
            </div>

        </div>

        <!-- Modal -->
        <!-- <p id="modalBtn" style="display:none;" data-toggle="modal" data-target="#myModal"></p>
        <div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-body" style="text-align:center;">
                        {{modalText}}
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger text-white" data-dismiss="modal">닫기</button>
                    </div>
                </div>
            </div>
        </div>       -->
    </div>
</template>

<script>
import http from '../http-common';
import $ from "jquery"
export default {
    data() {
        return {
            IdButton: true,
            InterestButton: false,
            inputIdData: "", // 검색창에 입력한 내용
            inputInterestData: "",
            resultIds: [], // 검색창에 입력한 내용과 같은 아이디 찾기
            resultInterests: [],
            idErrorMsg: "",
            interestErrorMsg: "",
            searchData: [],
            search: true,
            modal : false,
            modalText:"",
        }
    },
    methods: {
        IdClick() {
            this.IdButton = true;
            this.InterestButton = false;
        },
        InterestClick() {
            this.IdButton = false;
            this.InterestButton = true;
        },
        getIds() {
            var searchIdData = new RegExp(this.inputIdData);
            http
                .get('/user/searchByUserId/' + this.inputIdData)
                .then((res) => {
                    console.log(res)
                    if (res.data.resmsg == "아이디 검색 성공") {
                        for (var i=0; i<res.data.resValue.length; i++) {
                            var id = res.data.resValue[i].user_id
                            if (searchIdData.test(id) === true) {
                                this.resultIds.push({
                                    user_id: id,
                                    description: res.data.resValue[i].description,
                                    profile_url: res.data.resValue[i].profile_url,
                                    profile_filter: res.data.resValue[i].profile_filter,
                                })
                            }
                        }
                    } else if (res.data.resmsg == "아이디 검색 실패") {
                        this.idErrorMsg = "아이디가 일치하는 친구가 없습니다."
                    }
                })
                    .catch(() => {
                        this.errored = true;
                    })
                    .finally(() => (this.loading = false));
        },
        searchInput() {
            var text = document.getElementById('itrlone').value;
            if (text == "") {
                return;
            }
            if (this.searchData.indexOf(text)) {
                this.searchData.push(text)
            }
            if (this.searchData === []) {
                this.search = true;
            } else {
                this.search = false;
            }
            this.inputInterestData = ""
        },
        resetData() {
            this.searchData = [],
            this.resultInterests = []
            this.interestErrorMsg = ""
            this.search = true;
        },
        getInterests() {
            this.loading = true;
            http
                .post('/user/searchByInterest', this.searchData)
                .then((res) => {
                    console.log(res)
                    if (res.data.resmsg == "관심사 검색 성공") {
                        this.interestErrorMsg = ""
                        this.resultInterests = []
                        for (var i = 0; i < res.data.resValue.length; i++) {
                            this.resultInterests.push({
                                user_id: res.data.resValue[i].user_id,
                                interest: res.data.resValue[i].interest,
                                description: res.data.resValue[i].description,
                                profile_url: res.data.resValue[i].profile_url,
                                profile_filter: res.data.resValue[i].profile_filter,
                            })
                        }
                    } else {
                        this.interestErrorMsg = "관심사가 일치하는 친구가 없습니다.";
                        this.resultInterests = []
                    }
                })
                    .catch(() => {
                        this.errored = true;
                    })
                    .finally(() => (this.loading = false));
        },
    },
    watch: {
        inputIdData: function (inputId) {
            if (inputId === "") {
                this.resultIds = []
                this.idErrorMsg = ""
            } else {
                this.getIds()
                this.resultIds = []
                this.idErrorMsg = ""
            }
        },
    },
    mounted() {
        $('html').scrollTop(0);
    },
}
</script>