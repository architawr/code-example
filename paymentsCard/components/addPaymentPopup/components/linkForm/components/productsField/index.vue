<template lang="pug">
li.fields-table__item
  .fields-table__body.fields-table__body--full
    .fields-table__fields
      .fields-table__field
        .carrier-schedule__inner
          product-field(
            v-for="productRaw in productsRaw"
            :key="productRaw.productId"

            :productRaw="productRaw"
            :productsRaw="productsRaw"

            :products="products"
            :productsIsLoading="productsIsLoading"

            @edit="editProduct"
            @delete="deleteProduct"
          ).carrier-schedule__item

          v-btn(
            text

            @click="addProduct"
          ).order-card__action.carrier-schedule__button + Добавить товар
</template>

<script>
import productField from './components/productField';

export default {
  model: {
    prop: 'productsRaw',
    event: 'change'
  },

  props: {
    productsRaw: {
      type: Array,
      required: true
    },

    products: {
      type: Array,
      required: true
    },
    productsIsLoading: {
      type: Boolean,
      required: true
    }
  },

  methods: {
    addProduct() {
      if (this.hasEmptyProduct()) return;

      const emptyProduct = {
        productId: null,
        quantity: 1,
        price: null,
        discount: null,
        tax: null
      };

      this.productsRaw.push(Object.assign({}, emptyProduct));

      this.$emit('add', this.productsRaw);
      this.$emit('change', this.productsRaw);

      this.$forceUpdate();
    },

    hasEmptyProduct() {
      return !!this.productsRaw.find(product => !product.productId);
    },

    editProduct(item) {
      this.$emit('edit', item);
      this.$emit('change', this.productsRaw);
    },
    deleteProduct(item) {
      const productIndex = this.productsRaw.indexOf(item);

      this.productsRaw.splice(productIndex, 1);

      this.$emit('delete', item);
      this.$emit('change', this.productsRaw);
    }
  },

  components: {
    productField
  }
};
</script>

<style lang="scss" scoped>
.carrier-schedule {
  &__inner {
    display: flex;
    flex-wrap: wrap;
    margin: 0 -8px;
  }

  &__item {
    max-width: 256px;
    width: 100%;
    margin: 8px;
  }

  &__button {
    max-width: 256px;
    width: 100%;
    padding: 18px 16px !important;
    margin: 8px !important;
  }
}
</style>
