<template lang="pug">
li.fields-table__item
  .fields-table__label Тип транзакции
  .fields-table__body
    .fields-table__fields
      .fields-table__field
        Let(
          :val="transactionType(payment.transactionType)"
          #default="{ val: transactionTypeReady }"
        )
          //- v-progress-linear(v-if="regionsIsLoading" :indeterminate="true").loader.loader--min-w.loader--full
          //- template(v-else)
          | {{ transactionTypeReady ? transactionTypeReady.name : payment.transactionType  }}
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  props: {
    payment: {
      type: Object,
      required: true
    }
  },

  computed: {
    ...mapGetters({
      transactionType: 'blocks/payments/getTransactionType'
    })
  }
};
</script>
