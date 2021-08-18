<template lang="pug">
sweet-modal(
  ref="paymentInfoPopup"

  width="720px"
  title="Информация о платеже"
)
  div(v-if="payment").fields-table
    ul.fields-table__list
      li.fields-table__item
        .fields-table__label ID платежа
        .fields-table__body
          .fields-table__fields
            .fields-table__field {{ payment.paymentId }}
      li.fields-table__item
        .fields-table__label Сумма платежа
        .fields-table__body
          .fields-table__fields
            .fields-table__field {{ payment.sumPaid }} {{ orderCurrencyLabel }}
      li.fields-table__item
        .fields-table__label Время платежа
        .fields-table__body
          .fields-table__fields
            .fields-table__field {{ payment.timePaid | formatDate('dd MMM, HH:mm:ss') }}

      payment-method-field(
        :payment="payment"
      )

      transaction-type-field(
        :payment="payment"
      )

      accounting-type-field(
        :payment="payment"
      )

      li.fields-table__item
        .fields-table__label ID кассы
        .fields-table__body
          .fields-table__fields
            .fields-table__field {{ payment.posId }}

      template(v-if="payment.paymentMethod === 'online' || payment.paymentMethod === 'acquiring'")
        li.fields-table__item
          .fields-table__label Эквайер платежа
          .fields-table__body
            .fields-table__fields
              .fields-table__field {{ payment.acquierId }}
        li.fields-table__item
          .fields-table__label ID транзакции эквайера
          .fields-table__body
            .fields-table__fields
              .fields-table__field {{ payment.acquierTransactionId }}
        li.fields-table__item
          .fields-table__label Статус транзакции эквайера
          .fields-table__body
            .fields-table__fields
              .fields-table__field {{ payment.acquierTransactionStatusId }}

      li.fields-table__item
        .fields-table__label ID чека
        .fields-table__body
          .fields-table__fields
            .fields-table__field {{ payment.receiptId }}
      li.fields-table__item
        .fields-table__label Сумма чека
        .fields-table__body
          .fields-table__fields
            .fields-table__field {{ payment.receiptAmount }}
      li.fields-table__item
        .fields-table__label Ссылка на чек
        .fields-table__body
          .fields-table__fields
            .fields-table__field
              a(
                v-if="payment.receiptLink"

                :href="payment.receiptLink"

                target="_blank"
              ).link.link--clickable {{ payment.receiptLink }}
      receipt-items-field(
        :payment="payment"
      )

</template>

<script>
import paymentMethodField from './components/paymentMethodField';
import transactionTypeField from './components/transactionTypeField';
import accountingTypeField from './components/accountingTypeField';
import receiptItemsField from './components/receiptItemsField';

import { mapState } from 'vuex';

import cloneDeep from 'lodash/cloneDeep';

export default {
  data: () => ({
    payment: null
  }),

  computed: {
    ...mapState({
      order: state => state.order.info
    }),

    orderCurrencyLabel() {
      const { uppercase } = this.$options.filters;

      return this.$t(
        uppercase('SETTINGS.CURRENCIES.' + this.order.currencyId + '.SHORT')
      );
    }
  },

  methods: {
    open(payment) {
      this.$refs.paymentInfoPopup.open();

      this.payment = cloneDeep(payment);
    },
    close() {
      this.$refs.paymentInfoPopup.close();
    }
  },

  components: {
    paymentMethodField,
    transactionTypeField,
    accountingTypeField,
    receiptItemsField
  }
};
</script>

<style lang="scss" scoped></style>
