<template lang="pug">
li.fields-table__item.fields-table__item--start
  .fields-table__label Товары в чеке
  .fields-table__body
    .fields-table__fields
      .fields-table__field
        receipt-item(
          v-for="item in receiptItems"
          :key="item.id"

          :item="item"
        )
</template>

<script>
import receiptItem from './components/receiptItem';

export default {
  props: {
    payment: {
      type: Object,
      required: true
    }
  },

  computed: {
    receiptItems() {
      return this.payment?.receiptItems || [];
    }
  },

  components: {
    receiptItem
  }
};
</script>
