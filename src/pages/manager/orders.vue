<template>
  <layout>
    <template #fixed>
      <div class="block">
        <DateRange class="block__item" @change="handleDate" />
        <Search
          class="block__item"
          :searchQuery="searchQuery"
          :types="searchTypes"
          v-model:type="searchType"
          v-model:searchQuery="searchQuery"
        />
      </div>
      <div class="block">
        <SelectYear :items="selectItems" name="selectYear" v-model="selectedItem" />
        <div class="btn-block">
          <UiButton color="purple" size="small" @click="showData">Показать</UiButton>

          <UiButton color="gray" size="small" @click="resetData">Сбросить</UiButton>
        </div>
      </div>
    </template>

    <div>
      <div class="block" v-if="orders">
        <StatusCard v-for="(order, index) in orders" :key="index"> №{{ order.id }} </StatusCard>
      </div>
      <Empty v-else />
    </div>
  </layout>
</template>

<script>
import layout from "../../views/layout/fixed-left-column.vue";
import DateRange from "../../ui/date-range/index.vue";
import Search from "../../components/shared/search/index.vue";
import SelectYear from "../../ui/select/index.vue";
import StatusCard from "../../ui/status-card/index.vue";
import Empty from "../../ui/empty/index.vue";
import UiButton from "../../ui/button/index.vue";

export default {
  name: "orders",

  components: {
    layout,
    DateRange,
    Search,
    SelectYear,
    StatusCard,
    Empty,
    UiButton
  },

  mounted() {
    this.showData();
  },

  data() {
    return {
      orders: null,
      searchQuery: "",
      searchType: "order_number",
      searchTypes: {
        order_number: { title: "Номер заказа", placeholder: "Введите номер заказа", searchQuery: "" },

        psid: { title: "Номер фотосессии", placeholder: "Введите номер фотосессии", searchQuery: "" },

        client_id: { title: "Клиент ID", placeholder: "Введите ID", searchQuery: "" },

        phone: { title: "Телефон", placeholder: "Введите номер телефона", searchQuery: "" },

        email: { title: "email", placeholder: "Введите email", searchQuery: "" }
      },
      selectItems: [
        { value: "2023", title: "2023" },
        { value: "2022", title: "2022" },
        { value: "2021", title: "2021" }
      ],

      selectedItem: "null",
      dateStart: "",
      dateFinish: ""
    };
  },

  methods: {
    handleDate(value) {
      if ("date_start" in value) {
        this.dateStart = this.formatTimestampToDate(value.date_start);
      }

      if ("date_finish" in value) {
        this.dateFinish = this.formatTimestampToDate(value.date_finish);
      }
    },

    formatTimestampToDate(timestamp) {
      const date = new Date(timestamp * 1000);
      const year = date.getFullYear();
      const month = ("0" + (date.getMonth() + 1)).slice(-2);
      const day = ("0" + date.getDate()).slice(-2);

      return `${year}${month}${day}`;
    },

    async showData() {
      try {
        const queryParams = {
          date_start: this.dateStart,
          date_finish: this.dateFinish,
          search_type: this.searchType,
          search_value: this.searchQuery,
          year: this.selectedItem
        };

        const filteredParams = Object.fromEntries(Object.entries(queryParams).filter(([value]) => value !== undefined));

        const data = await $fetch(`/method/orders.getTest`, {
          query: filteredParams,
          method: "GET"
        });

        const { response } = JSON.parse(data);
        this.orders = response?.data?.orders;
      } catch (error) {
        console.error("There has been a problem with your fetch operation:", error.message);
      }
    },

    resetData() {
      this.orders = null;
      this.dateStart = "null";
      this.dateFinish = "null";
      this.searchQuery = "null";
      this.selectedItem = null;

      this.showData();
    }
  }
};
</script>

<style lang="scss" scoped>
.block {
  padding: 16px;
  margin-bottom: 15px;

  &__item {
    margin-bottom: 15px;
  }
}

.btn-block {
  display: flex;
  flex-wrap: nowrap;
  margin-top: 15px;
}

.button {
  width: 50%;
  padding: 10px 0;
  margin-right: 15px;

  &:nth-child(2n) {
    margin-right: 0;
  }
}
</style>
