<template>
  <section>
    <the-title
      title="Rounds"
      icon="sword-cross"
      class="mb-4"
      :props="{ teamId, gameId }"
      component="round-add-dialog"
    />
    <div v-if="showTable">
      <v-tabs v-model="tab" background-color="primary" centered dark icons-and-text>
        <v-tabs-slider color="secondary" />
        <v-tab v-for="(tabItem, i) in tabs" :key="tabItem.name" :href="`#tab-${i}`">
          <span class="mt-2">{{ tabItem.name }}</span>
          <v-icon>{{ `mdi-${tabItem.icon}` }}</v-icon>
        </v-tab>
      </v-tabs>
      <v-tabs-items v-model="tab">
        <v-tab-item v-for="(component, i) in tabComponents" :key="i" :value="`tab-${i}`">
          <component :is="component" :team="team" :rounds="rounds" />
        </v-tab-item>
      </v-tabs-items>
    </div>
  </section>
</template>

<script>
import TheTitle from "@/components/TheTitle";
import MainTable from "@/components/MainTable";
import ChartBars from "@/components/ChartBars";
import rounds from "@/store/modules/rounds";
import { mapActions, mapGetters } from "vuex";

export default {
  components: {
    ChartBars,
    MainTable,
    TheTitle
  },
  props: {
    teamId: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      tab: null,
      gameId: this.$route.query.gameId,
      tabs: [
        {
          name: "Table",
          href: "tab-1",
          icon: "table-large"
        },
        {
          name: "Statistics",
          href: "tab-2",
          icon: "chart-bar"
        }
      ],
      tabComponents: ["the-table", "chart-bars"]
    };
  },
  computed: {
    ...mapGetters("teams", ["getTeam"]),
    ...mapGetters("games", ["getGame"]),
    ...mapGetters("rounds", ["getRounds"]),
    game() {
      return this.getGame(this.gameId);
    },
    team() {
      return this.getTeam(this.teamId);
    },
    rounds() {
      const query = { teamId: this.teamId, gameId: this.gameId };
      const rounds = this.getRounds(query);
      if (rounds) rounds.forEach(round => (round[round.winner] = "VICTORY"));
      return rounds;
    },

    showTable() {
      return this.rounds?.length;
    }
  },
  created() {
    this.loadData();
    this.setBackTitle(`${this.team.name}: ${this.game.name}`);
  },
  beforeDestroy() {
    this.setBackTitle();
  },
  methods: {
    ...mapActions(["setBackTitle"]),
    ...mapActions("rounds", ["loadRounds"]),
    loadData() {
      const isRounds = this.$store.hasModule("rounds");
      isRounds || this.$store.registerModule("rounds", rounds);
      this.loadRounds();
    }
  }
};
</script>
