<template>
  <v-row class="items px-2 py-1">
    <v-col class="pb-0 mb-2">
      <v-autocomplete
        dense
        clearable
        auto-select-first
        outlined
        color="indigo"
        label="Table"
        v-model="table"
        :items="tables"
        item-text="table"
        item-value="table"
        background-color="white"
        no-data-text="Table not found"
        hide-details
        :filter="customFilter"
        :disabled="readonly"
      ></v-autocomplete>
    </v-col>

    <v-col class="pb-0 mb-2">
      <v-autocomplete
        dense
        clearable
        auto-select-first
        outlined
        color="indigo"
        label="Servant"
        v-model="servant"
        :items="servants"
        item-text="name1"
        item-value="name1"
        background-color="white"
        no-data-text="Servant not found"
        hide-details
        :filter="customServantFilter"
        :disabled="readonly"
      ></v-autocomplete>
    </v-col>
  </v-row>
</template>

<script>
import { evntBus } from "../../bus";
export default {
  data: () => ({
    pos_profile: "",
    tables: [],
    table: "",
    servants: [],
    servant: "",
    readonly: false,
  }),

  methods: {
    get_table_names() {
      const vm = this;
      if (vm.pos_profile.posa_local_storage && localStorage.table_storage) {
        vm.tables = JSON.parse(localStorage.getItem("table_storage"));
      }

      frappe.call({
        method: "posawesome.posawesome.api.posapp.get_table_names",
        args: { pos_profile: vm.pos_profile.pos_profile.name },
        callback: function (r) {
          if (r.message) {
            vm.tables = r.message;
            console.log(vm.tables);
            console.log("loadTables");
            if (vm.pos_profile.posa_local_storage) {
              localStorage.setItem("table_storage", "");
              localStorage.setItem("table_storage", JSON.stringify(r.message));
            }
          }
        },
      });
    },
    get_servant_names() {
      const vm = this;
      if (vm.pos_profile.posa_local_storage && localStorage.servant_storage) {
        vm.servants = JSON.parse(localStorage.getItem("servant_storage"));
      }

      frappe.call({
        method: "posawesome.posawesome.api.posapp.get_servant_names",
        args: { pos_profile: vm.pos_profile.pos_profile.name },
        callback: function (r) {
          if (r.message) {
            vm.servants = r.message;
            console.log(vm.servants);
            console.log("loadServant");
            if (vm.pos_profile.posa_local_storage) {
              localStorage.setItem("servant_storage", "");
              localStorage.setItem(
                "servant_storage",
                JSON.stringify(r.message)
              );
            }
          }
        },
      });
    },
    customFilter(item, queryText, itemText) {
      const textOne = item.table ? item.table.toLowerCase() : "";

      const searchText = queryText.toLowerCase();

      return textOne.indexOf(searchText) > -1;
    },
    customServantFilter(item, queryText, itemText) {
      const textOne = item.name1 ? item.name1.toLowerCase() : "";

      const searchText = queryText.toLowerCase();

      return textOne.indexOf(searchText) > -1;
    },
  },

  computed: {},

  created: function () {
    this.$nextTick(function () {
      evntBus.$on("register_pos_profile", (pos_profile) => {
        this.pos_profile = pos_profile;
        this.get_table_names();
        this.get_servant_names();
      });
      evntBus.$on("set_table", (table) => {
        this.table = table;
      });
      evntBus.$on("set_servant", (servant) => {
        this.servant = servant;
      });
    });
  },
};
</script>
