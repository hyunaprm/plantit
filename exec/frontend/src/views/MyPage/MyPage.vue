<template>
  <div class="content">
    <div class="head">
      <img :src="Planetimg" />
    </div>
    <div class="division"></div>
    <div class="top-button">
      <div id="getsurveyresult" @click="getSurveyResult()" v-if="isSurvey">
        성향 및 추천식물 다시보기
      </div>
      <div id="gotosurvey" @click="moveToSurvey()" v-else>
        나와 맞는 식물 찾기
      </div>
      <div id="logout" @click="logOut()">LOGOUT</div>
    </div>
    <ProfileCard :myprofile="this.myprofile" />
    <div id="planet-division-bar"></div>
    <div class="planet-title">
      <div id="mypage">MY PLANET</div>
      <div id="care">레벨을 높이고 행성을 푸르게 만들어주세요</div>
    </div>
    <MyPlanet v-bind:level="this.myprofile.level.level" />
    <div id="planet-division-bar"></div>
    <div style="padding-bottom: 1%"></div>
    <div class="planet-title">
      <div id="mypage">MY REWARD</div>
      <div id="care">미션을 수행하고 뱃지를 모아보세요</div>
    </div>
    <div class="myreward">
      <MyReward
        v-for="(myreward, idx) in reward_status"
        :key="idx"
        :myreward="myreward"
        :index="idx"
      />
    </div>
    <div id="deleteUser" @click="deleteUser()"><span>회원탈퇴</span></div>
    <div id="planet-division-bar"></div>
    <div id="reward-bottom-full"></div>
    <div style="padding-bottom: 75px"></div>
  </div>
</template>

<script>
import axios from "axios";
import Planetimg from "@/assets/img/PLANET.png";
import ProfileCard from "@/components/MyPage/ProfileCard.vue";
import MyPlanet from "@/components/MyPage/MyPlanet.vue";
import MyReward from "@/components/MyPage/MyReward.vue";

const SERVER_URL = process.env.VUE_APP_SERVER_URL;
export default {
  name: "MyPage",
  data: () => {
    return {
      Planetimg: Planetimg,
      myprofile: {
        level: {
          level: Number,
          level_pc: Number,
          max_reward_cnt: Number,
          reward_cnt: Number,
          title: String,
        },
        reward: {
          reward1: Number,
          reward2: Number,
          reward3: Number,
          reward4: Number,
          reward5: Number,
          reward6: Number,
          reward7: Number,
          reward8: Number,
          reward9: Number,
          reward10: Number,
          reward11: Number,
          reward12: Number,
        },
      },
      reward_status: [],
      isSurvey: false,
    };
  },
  components: {
    ProfileCard,
    MyPlanet,
    MyReward,
  },
  methods: {
    logOut() {
      localStorage.removeItem("email");
      localStorage.removeItem("name");
      localStorage.removeItem("jwt");
      localStorage.removeItem("user_number");
      localStorage.removeItem("is_survey");
      this.$swal.fire({
        icon: "success",
        title: "로그아웃 되었습니다.",
      });
      this.$router.push({ name: "Home" }).catch((error) => {
        if (error.name === "NavigationDuplicated") {
          location.reload();
        }
      });
    },
    getSurveyResult() {
      this.$router.push({ name: "SurveyResult" }).catch((error) => {
        if (error.name === "NavigationDuplicated") {
          location.reload();
        }
      });
    },
    moveToSurvey() {
      this.$router.push({ name: "SurveyProgress" }).catch((error) => {
        if (error.name === "NavigationDuplicated") {
          location.reload();
        }
      });
    },
    deleteUser() {
      this.$swal
        .fire({
          title: "정말 지구 환경을 포기하고 탈퇴하시겠습니까?",
          icon: "warning",
          showCancelButton: true,
          cancelButtonText: "아뇨, 참아볼게요",
          confirmButtonText: "네. 미안해요 :(",
          reverseButtons: true,
        })
        .then((result) => {
          if (result.isConfirmed) {
            axios
              .delete(`${SERVER_URL}/delete/`, {
                data: {
                  user_number: localStorage.getItem("user_number"),
                },
              })

              .then(({ data }) => {
                let msg = "탈퇴에 실패하였습니다.";
                if (data.message === "success") {
                  localStorage.removeItem("email");
                  localStorage.removeItem("name");
                  localStorage.removeItem("jwt");
                  localStorage.removeItem("user_number");
                  localStorage.removeItem("is_survey");
                  this.$router.push({ name: "Home" }).catch((error) => {
                    if (error.name === "NavigationDuplicated") {
                      location.reload();
                    }
                  });
                  location.reload();
                } else {
                  alert(msg);
                  location.reload();
                }
              })
              .catch(() => {
                alert("서버와 통신할 수 없습니다.");
              });
          }
        });
    },
    getProfile() {
      axios
        .post(`${SERVER_URL}/mypage/`, {
          user_number: localStorage.getItem("user_number"),
        })
        .then((response) => {
          this.myprofile = response.data;
          this.reward_status = Object.values(this.myprofile.reward);
        })
        .catch(() => {
          alert("결과를 가져오는 중에 오류가 발생했습니다");
          this.$router.push({ name: "Home" }).catch((error) => {
            if (error.name === "NavigationDuplicated") {
              location.reload();
            }
          });
        });
    },
  },
  created() {
    const token = localStorage.getItem("jwt");
    if (token == null) {
      this.$swal.fire({
        icon: "error",
        title: "로그인 하고 이용해주세요.",
      });
      this.$router.push({ name: "Home" }).catch((error) => {
        if (error.name === "NavigationDuplicated") {
          location.reload();
        }
      });
    } else {
      this.getProfile();
      if (localStorage.getItem("is_survey") == 1) {
        this.isSurvey = true;
      }
    }
  },
};
</script>