<template>
<fragment>

  <div class="d-flex pa-0 align-center">
    <h1>Sessions</h1>
    <v-spacer/>
    <v-spacer/>
  </div>

  <v-card class="mt-2">
    <v-app-bar flat color="transparent">
      <v-toolbar-title></v-toolbar-title>
    </v-app-bar>

    <v-divider></v-divider>

    <v-card-text class="pa-0">
      <v-data-table :headers="headers" :items=listSessions item-key="uid" :sort-by="['started_at']" :sort-desc="[true]" :items-per-page="10" :server-items-length='this.numberSessions' :options.sync='pagination'>
        <template v-slot:item.active="{ item }">
          <v-icon color="success" v-if="item.active">check_circle</v-icon>
          <v-tooltip bottom v-else>
            <template #activator="{ on }">
              <v-icon v-on="on">check_circle</v-icon>
            </template>
            <span>active {{ item.last_seen | moment("from", "now") }}</span>
          </v-tooltip>
        </template>

        <template v-slot:item.device="{ item }">
          <router-link :to="{ name: 'detailsDevice', params: { id: item.device.uid } }">
            {{ item.device.name }}
          </router-link>
        </template>

        <template v-slot:item.username="{ item }">
          <v-tooltip bottom v-if="!item.authenticated">
            <template #activator="{ on }">
            </template>
            <span>Unauthorized</span>
          </v-tooltip>
          {{ item.username }}
        </template>

        <template v-slot:item.authenticated="{ item }">
          <v-tooltip bottom>
            <template #activator="{ on }">
              <v-icon v-on="on" :color="item.active ? 'success' : ''" size="" v-if="item.authenticated">mdi-shield-check</v-icon>
              <v-icon v-on="on" color="error" size="" v-else>mdi-shield-alert</v-icon>
            </template>
            <span v-if="item.authenticated">User has been authenticated</span>
            <span v-else>User has not been authenticated</span>
          </v-tooltip>
        </template>

        <template v-slot:item.ip_address="{ item }">
          <code>{{ item.ip_address }}</code>
        </template>

        <template v-slot:item.started="{ item }">
          {{ item.started_at | moment("ddd, MMM Do YY, h:mm:ss a")}}
        </template>

        <template v-slot:item.last_seen="{ item }">
          {{ item.last_seen | moment("ddd, MMM Do YY, h:mm:ss a")}}
        </template>

        <template v-slot:item.actions="{ item }">
          <v-icon class="icons" @click="detailsSession(item)">
            info
          </v-icon>

          <v-icon class="icons ml-1" v-if="item.active" @click="closeSession(item)">
            mdi-close-circle
          </v-icon>
        </template>
      </v-data-table>
    </v-card-text>
  <v-snackbar v-model="sessionSnack" :timeout=3000>Session closed</v-snackbar>
  </v-card>

</fragment>
</template>

<script>
export default {
  name: "SessionList",

  data() {
    return {
      sessionSnack: false,
      numberSessions: 0,
      listSessions: [],
      data: [],
      pagination: {},

      headers: [
        {
          text: "Active",
          value: "active",
          align: "center"
        },
        {
          text: "Device",
          value: "device",
          align: "center"
        },
        {
          text: "Username",
          value: "username",
          align: "center"
        },
        {
          text: "Authenticated",
          value: "authenticated",
          align: "center"
        },
        {
          text: "IP Address",
          value: "ip_address",
          align: "center"
        },
        {
          text: "Started",
          value: "started",
          align: "center"
        },
        {
          text: "Last Seen",
          value: "last_seen",
          align: "center"
        },
        {
          text: "Actions",
          value: "actions",
          align: "center"
        }
      ]
    };
  },

  watch: {
    pagination: {
      async handler () {
        const rowsPerPage = this.pagination.itemsPerPage
        const pageNumber = this.pagination.page

        this.data =  [{'per_page': rowsPerPage, 'page':pageNumber}]

        await this.$store.dispatch("sessions/fetch", this.data[0]);
        this.listSessions = this.$store.getters["sessions/list"];
        this.numberSessions = this.$store.getters["sessions/getNumberSessions"]; 
      },
      deep: true
    },
  },

  methods: {
    detailsSession(session) {
      this.$router.push("/session/" + session.uid);
    },
    async closeSession(session) {
      if (confirm("Are you sure?", session)) {
        this.$store.dispatch("sessions/close", session);
        this.sessionSnack = true;

        await this.$store.dispatch("sessions/fetch");
        this.listSessions = this.$store.getters["sessions/list"];
      }
    },
  }
};
</script>
