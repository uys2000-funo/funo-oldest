<template>
  <div class="column no-wrap justify-center items-center content-center">
    <div class="col-4">
      <div class="reg">
        <register-wheel
          :style="`display: ${page != -4 && page != 4 ? '' : 'none'}`"
          :page="page"
        />
        <tag-order v-if="page == -4 || page == 4" :imgs="imgs" />
      </div>
      <div class="bec">
        <q-btn round flat fab-mini @click="goBack">
          <q-icon>
            <img
              :src="require('@/assets/images/icons/backArrow.svg')"
              alt="icon"
            />
          </q-icon>
        </q-btn>
      </div>
    </div>
    <div class="col-7">
      <router-view
        :page="page"
        :right="right"
        :uWatch="uWatch"
        ref="r"
        :img="img"
      />
    </div>
    <div class="col-1">
      <q-btn
        v-if="page != 0 && page != -4 && page != 4"
        class="btn"
        @click="goNext"
      >
        <div>Devam</div>
      </q-btn>
      <q-btn
        v-if="page == -4 || page == 4"
        :disable="!right"
        class="btn"
        @click="registerEvent"
      >
        <div>Kaydol</div>
      </q-btn>
    </div>
  </div>
</template>
<script>
import registerWheel from "@/components/register/compRegisterWheel.vue";
import tagOrder from "@/components/register/compTagOrder.vue";
import { registerCheck } from "@/services/core/main";
import { registerFunction } from "../services/firebase/login";
import { register } from "@/store/register";
export default {
  components: {
    registerWheel,
    tagOrder,
  },
  data() {
    return {
      img: null,
      page: 0,
      right: false,
      uWatch: false,
      register: register(),
      imgs: [],
      user: {},
    };
  },
  methods: {
    updateImgs: function (val) {
      if (this.imgs.find((i) => i == val))
        this.imgs = this.imgs.filter((i) => {
          return i != val;
        });
      else this.imgs.push(val);
    },
    setImage: function (img) {
      console.log(img);
      this.img = img;
    },
    abs: function (val) {
      return Math.abs(val);
    },
    setPage: function (value) {
      this.page = value;
    },
    goNext: function () {
      this.page < 0 ? (this.page -= 1) : (this.page += 1);
    },
    goBack: function () {
      this.page < 0 ? (this.page += 1) : (this.page -= 1);
      if (this.page == 0) this.$router.go(-1);
    },
    registerEvent: function () {
      this.uWatch = true;
      // To wait watch function in RegsterP ad registerC pages
      setTimeout(() => {
        this.register.user.tags = this.imgs;
        registerCheck(this.register.user).then((r) => {
          if (r)
            registerFunction(this.register.user, this.img).then((res) => {
              if (res) this.$router.push("/login/");
              else alert("Some Problems");
            });
        });
      }, 10);
    },
  },
  provide() {
    return {
      updateImgs: this.updateImgs,
      setPage: this.setPage,
      setRight: () => (this.right = !this.right),
      setImage: (img) => this.setImage(img),
    };
  },
};
</script>
<style scoped>
.reg {
  height: 100%;
  margin: auto;
}
.bec {
  position: fixed;
  left: 2vw;
  top: 1vh;
}
.btn {
  background: #ff7f00;
  border-radius: 20px;
  width: 75vw;
}
</style>
