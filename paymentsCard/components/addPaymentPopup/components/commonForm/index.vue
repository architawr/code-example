<template lang="pug">
v-form(
  ref="defaultForm"

  v-model="valid"

  @submit.prevent="addPayment"
).fields-table
  ul.fields-table__list
    li.fields-table__item
      .fields-table__label Сумма
      .fields-table__body
        .fields-table__fields
          .fields-table__field
            v-text-field(
              flat
              type="number"
              min="1"

              v-model.number="payment.sumPaid"

              :suffix="orderCurrencyLabel"
              single-line

              :rules="[rules.required]"
            )

    li.fields-table__item
      .fields-table__label Метод
      .fields-table__body
        .fields-table__fields
          .fields-table__field
            v-select(
              flat

              :items="paymentMethods"
              v-model="payment.paymentMethod"

              item-text="name"
              item-value="id"

              single-line

              :rules="[rules.required]"
            )

    template(v-if="payment.paymentMethod === 'online' || payment.paymentMethod === 'acquiring'")
      li.fields-table__item
        .fields-table__label Эквайер
        .fields-table__body
          .fields-table__fields
            .fields-table__field
              v-select(
                flat

                :items="acquiers"
                v-model="payment.acquierId"

                single-line

                :rules="[rules.required]"
              )

      li.fields-table__item
        .fields-table__label ID транзакции
        .fields-table__body
          .fields-table__fields
            .fields-table__field
              v-text-field(
                flat
                type="text"

                v-model="payment.acquierTransactionId"

                single-line

                :rules="[rules.required]"
              )

    li.fields-table__item
      .fields-table__label Применяется к
      .fields-table__body
        .fields-table__fields
          .fields-table__field
            v-select(
              flat
              multiple

              :items="products"
              v-model="payment.productIds"
              :loading="productsIsLoading"

              item-text="title"
              item-value="productId"

              single-line

              :rules="[rules.requiredArray]"
            )
</template>

<script>
import { mapState } from 'vuex';

import { rules } from '@/utils/forms.js';

export default {
  data: () => ({
    valid: false,

    rules
  }),

  props: {
    payment: {
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

  computed: {
    ...mapState({
      order: state => state.order.info,

      paymentMethods: state => state.blocks.payments.methods,
      acquiers: state => state.blocks.payments.acquiers
    }),

    orderCurrencyLabel() {
      const { uppercase } = this.$options.filters;

      return this.$t(
        uppercase('SETTINGS.CURRENCIES.' + this.order.currencyId + '.SHORT')
      );
    }
  },

  methods: {
    validate() {
      return this.$refs.defaultForm.validate();
    },

    resetValidation() {
      this.$refs.defaultForm.resetValidation();
    }
  }
};
</script>
