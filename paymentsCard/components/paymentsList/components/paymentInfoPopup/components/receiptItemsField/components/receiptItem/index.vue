<template lang="pug">
.product
  .product__header {{ item.name }}
  .product__body
    .product__fields
      .product__field
        .product__field-label Кол-во:
        .product__field-value {{ item.quantity }}
      .product__field
        .product__field-label Цена:
        .product__field-value {{ item.price }}
      .product__field
        .product__field-label Стоимость:
        .product__field-value {{ item.sum }}

      .product__field
        .product__field-label НДС, в %:
        .product__field-value {{ item.vatRate }}
      .product__field
        .product__field-label НДС:
        .product__field-value {{ item.vatValue }}
      .product__field
        .product__field-label Налоговый ID:
        .product__field-value {{ item.merchantTaxId }}


</template>

<script>
export default {
  props: {
    item: {
      type: Object,
      required: true
    }
  },

  computed: {},

  methods: {}
};
</script>

<style lang="scss" scoped>
.product {
  max-width: 256px;

  display: flex;
  flex-direction: column;
  background-color: var(--v-secondary-lighten2);

  &::v-deep {
    .v-select.v-text-field input {
      width: 0 !important;
    }
  }

  &__header {
    background-color: var(--v-secondary-lighten1);
    padding: 8px 16px;
  }

  &__title {
    color: var(--v-secondary-darken2);
  }

  &__body {
    display: flex;
    flex-direction: column;
    flex: 1;
    padding: 8px 16px;
  }

  &__fields {
    flex: 1;
  }

  &__field {
    display: flex;

    &-label {
      margin-right: 8px;
    }
  }
}
</style>
