<template lang="pug">
.product
  .product__header
    v-select(
      :items="productsReady"
      :value="productRaw.productId"
      :loading="productsIsLoading"

      item-text="title"
      item-value="productId"

      label="Товар"
      height="20"

      single-line
      return-object

      :rules="[rules.required]"

      @change="onProductIdChange"
    )
  .product__body
    .product__fields
      .product__field
        price-field(
          v-model.number="productRaw.price"

          :maxValue="productRawMaxPrice"

          @change="onPriceChange"
        )

      .product__field
        v-text-field(
          type="number"
          :max="productsReadyCount"

          v-model.number="productRaw.quantity"

          name="quantity"
          label="Количество"
          height="20"
          suffix="шт"

          :rules="[rules.required, rules.onlyPositiveNumbers, rules.onlyFixedNumbers, productRawMaxQuantityRule]"

          @change="onQuantityChange"
        )

    v-btn(
      text
      block
      small

      color="error"

      @click="deleteProduct"
    ).product__delete удалить

</template>

<script>
import priceField from './components/priceField';

import { mapState } from 'vuex';

import { rules } from '@/utils/forms.js';

export default {
  data: () => ({
    rules
  }),

  props: {
    productsRaw: {
      type: Array,
      required: true
    },

    productRaw: {
      type: Object,
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

  created() {
    this.initialFetch();
  },

  computed: {
    ...mapState({
      isEditable: state => state.blocks.payments.fields.cart.isEditable,
      isEditableIsLoading: state => state.blocks.payments.fields.cart.isEditableIsLoading
    }),

    product() {
      return this.products.find(
        product => product.productId === this.productRaw.productId
      );
    },

    productsReady() {
      return this.products.filter(
        product =>
          !this.productsRaw?.find(
            productRaw => productRaw.productId === product.productId
          ) || this.productRaw.productId === product.productId
      );
    },
    productsReadyCount() {
      return this.productsReady.reduce(
        (acc, productReady) => acc + (productReady.quantity || 1),
        0
      );
    },

    productRawMaxPrice() {
      return this.product?.price * (this.productRaw.quantity || 1);
    },

    productRawMaxQuantityRule() {
      if (!this.productsReadyCount) return v => !!v;

      return v =>
        (v || 0) <= this.productsReadyCount ||
        `Количество не должно превышать ${this.productsReadyCount}`;
    }
  },

  methods: {
    async initialFetch() {
      await this.$store.cache.dispatch('blocks/payments/fields/cart/getInfo');
    },

    onProductIdChange(product) {
      const { productId, quantity, price, discount, tax } = product;

      this.$set(this.productRaw, 'productId', productId);
      this.$set(this.productRaw, 'quantity', quantity);
      this.$set(this.productRaw, 'price', price);
      this.$set(this.productRaw, 'discount', discount);
      this.$set(this.productRaw, 'tax', tax);

      this.$emit('edit', this.productRaw);
    },

    onPriceChange() {
      this.$emit('edit', this.productRaw);
    },
    onQuantityChange() {
      this.$emit('edit', this.productRaw);
    },

    deleteProduct() {
      this.$emit('delete', this.productRaw);
    }
  },

  components: {
    priceField
  }
};
</script>

<style lang="scss" scoped>
.product {
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
    margin-bottom: 4px;
  }

  &__delete {
    flex: 0 0 auto !important;
    margin: 0 !important;
  }
}
</style>
