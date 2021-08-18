<template lang="pug">
li.fields-table__item
  .fields-table__label Весь заказ
  .fields-table__body
    .fields-table__fields
      .fields-table__field.fields-table__field--x-small
        v-checkbox(
          :inputValue="isForAllOrder"

          :loading="productsIsLoading || servicesIsLoading"

          name="allOrder"
          height="20"
          single-line

          @change="onAllOrderChange"
        )
</template>

<script>
export default {
  model: {
    prop: 'isForAllOrder',
    event: 'change'
  },

  props: {
    isForAllOrder: {
      type: Boolean,
      required: true
    },

    productsIsLoading: {
      type: Boolean,
      required: true
    },
    servicesIsLoading: {
      type: Boolean,
      required: true
    }
  },

  methods: {
    onAllOrderChange(isForAllOrder) {
      this.$emit('change', isForAllOrder || false);
    }
  }
};
</script>
