<template>
  <div class="content">
    <div class="head" id="planit">
      <img class="planit_logo" :src="Planitimg" />
      <div class="planit_profile">
        <img
          @click="goMyPage()"
          v-if="mypage_list.profile_image"
          :src="mypage_list.profile_image"
        />
        <img @click="goMyPage()" v-else :src="getDefaultProfileImg()" />
      </div>
    </div>
    <div class="division" id="planit_division"></div>
    <div class="planit_myplant">
      <MainMyplant :myplant_list="myplant_list['myplant_list']" />
    </div>
    <div class="planit_divisionbar"></div>
    <div class="planit_foryou">
      <MainForyou :recommend_list="recommend_list" />
    </div>
    <div class="planit_divisionbar"></div>
    <div class="planit_magazine">
      <MainMagazine />
      <div style="padding-bottom: 75px"></div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

import Planitimg from "@/assets/img/PLANIT.png";

import MainMyplant from "@/components/PlanIt/MyPlant.vue";
import MainForyou from "@/components/PlanIt/ForYou.vue";
import MainMagazine from "@/components/PlanIt/MagazineCard.vue";

const SERVER_URL = process.env.VUE_APP_SERVER_URL;

export default {
  name: "Main",
  data: () => {
    return {
      Planitimg: Planitimg,

      myplant_list: "",
      recommend_list: "",
      mypage_list: "",
    };
  },
  components: {
    MainMyplant,
    MainForyou,
    MainMagazine,
  },
  methods: {
    getDefaultProfileImg() {
      return require(`@/assets/img/profile/basic_profile_img.png`);
    },
    goMyPage() {
      document.documentElement.scrollTop = 0;
      this.$router.push({ name: "MyPage" }).catch((error) => {
        if (error.name === "NavigationDuplicated") {
          location.reload();
        }
      });
    },
    goToSlide(index) {
      this.$refs.mycarousel.goSlide(index);
    },

    async getmyplant() {
      await axios
        .post(`${SERVER_URL}/myplant/`, {
          user_number: localStorage.getItem("user_number"),
        })

        .then((response) => {
          this.myplant_list = response.data;
        })
        .catch(() => {
          alert("서버와 통신할 수 없습니다.");
        });

      await axios
        .post(`${SERVER_URL}/detail/`, {
          user_number: localStorage.getItem("user_number"),
        })
        .then((response) => {
          this.recommend_list = response.data;
        })
        .catch(() => {
          alert("서버와 통신할 수 없습니다.");
        });

      await axios
        .post(`${SERVER_URL}/mypage/`, {
          user_number: localStorage.getItem("user_number"),
        })

        .then((response) => {
          this.mypage_list = response.data;
        })
        .catch(() => {
          alert("서버와 통신할 수 없습니다.");
        });
    },
  },

  // Mount 이전에, localStorage값이 1인 경우 ( 로그인 안해서 storage에 정보가 들어가지 않은 경우 ) Intro로 돌아가라.
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
      this.getmyplant();
    }
  },
};
</script>
