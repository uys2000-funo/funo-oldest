<template>
  <div v-if="checkTags" class="q-mb-md a">
    <!--Upper Texts and Settings Button-->
    <div class="row no-wrap justify-between items-start content-start">
      <span class="upper-texts">
        <router-link
          :to="{
            path:
              event.owner == user.ID
                ? `/app/user/share`
                : `/app/profile/${event.owner}/share`,
          }"
        >
          {{ event.ownerName }}
        </router-link>
      </span>
      <div
        style="width: 50%"
        class="row no-wrap justify-end items-end content-center"
      >
        <span class="upper-texts" style="max-width: 90%">
          {{ event.name }}
        </span>
        <!--Seetings Button-->
        <q-icon
          v-ripple
          size="xs"
          name="more_vert"
          @click="pages.openEventSettinsg(event)"
        />
      </div>
    </div>
    <!--Image-->
    <div
      class="img"
      :style="`background-image: url('${require('@/assets/images/loading.gif')}'); ${eventImage}`"
    ></div>
    <!--Bottom Informations-->
    <!--First Line-->
    <div class="row wrap justify-between items-center content-center">
      <span class="col-4">
        {{ event.startDate.time }}-{{ event.endDate.time }}
      </span>
      <span class="col-4 text-center q-my-sm">
        {{ event.price == 0 ? "ücretsiz" : event.price }}
      </span>
      <comp-participants-vue class="col-4 text-right" :userIDs="event.users" />
    </div>
    <!--Second Line-->
    <div class="row wrap justify-between items-center content-center">
      <span class="col-4 upper-texts" style="max-width: 75%">
        {{ event.app.text }}
      </span>
      <q-btn
        class="col-4 text-center bg-primary"
        :label="checkEvent ? 'Vazgeç' : 'katıl'"
        dense
        @click="checkEvent ? exitEvent(event) : joinEvent(event)"
      />
      <span class="col-4 text-right">
        <router-link :to="{ path: `/app/main/events/event/${event.eID}` }">
          Dahası
        </router-link>
      </span>
    </div>
  </div>
</template>

<script>
import { getImgStorage } from "@/services/firebase/event";
import { chekUserEventJoinStatus } from "@/services/core/main";
import compParticipantsVue from "./compEvent/compParticipants.vue";
import { pages } from "@/store/pages";
import { user } from "@/store/user";
import { events } from "@/store/events";
import {
  exitEventDB,
  joinEventDB,
  updatePopEventDB,
} from "@/services/core/events";
export default {
  props: ["event", "tags"],
  components: {
    compParticipantsVue,
  },
  data() {
    return {
      pages: pages(),
      eventImageUrl: "",
      user: user(),
    };
  },
  computed: {
    eventImage: function () {
      return `background-image: url("${this.eventImageUrl}"); `;
    },
    checkTags: function () {
      const tl = this.tags.length;
      if (tl != 0) {
        let c = 0;
        this.tags.forEach((i) => (this.event.tags[i] ? (c += 1) : 0));
        if (c == tl) return true;
        else return false;
      } else return true;
    },
    checkEvent: function () {
      return chekUserEventJoinStatus(this.user.user, this.event.eID);
    },
  },
  methods: {
    addEventToUser: user().addEventToUser,
    addUserToEvent: events().addUserToEvent,
    removeEventFromUser: user().removeEventFromUser,
    removeUserFromEvent: events().removeUserFromEvent,
    joinEventDB: joinEventDB,
    exitEventDB: exitEventDB,
    updatePopEvent: events().updatePopEvent,
    updatePopEventDB: updatePopEventDB,
    getImg: function () {},
    joinEvent: function (event) {
      this.addEventToUser(event.eID);
      this.addUserToEvent(event.eID, this.user.ID);
      this.joinEventDB(event.eID, this.user.ID);
      const [eNew, eOld] = this.updatePopEvent(event.eID);
      console.log([eNew, eOld]);
      if (eNew) this.updatePopEventDB(eNew, eOld);
    },
    exitEvent: function (event) {
      this.removeEventFromUser(event.eID);
      this.removeUserFromEvent(event.eID, this.user.ID);
      this.exitEventDB(event.eID, this.user.ID);
    },
  },
  mounted() {
    getImgStorage(`E/${this.event.eID}/imgs/img0`).then((res) => {
      this.eventImageUrl = res;
    });
  },
};
</script>
<style scoped>
.upper-texts {
  max-width: 45%;
  display: inline-block;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.img {
  height: 200px;
  border-radius: 10px;
  margin: auto;
  background-position: center;
  background-size: cover;
}
.q-btn {
  border-radius: 10px;
  width: 25%;
}
.a {
  background: rgba(245, 247, 251, 0.01);
  box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25), 0px 4px 4px rgba(0, 0, 0, 0.25);
  border-radius: 10px;
  padding: 10px 15px;
  max-width: 100vw;
}
a {
  color: black;
  text-decoration: none;
}
a:visited {
  color: black;
}
</style>
