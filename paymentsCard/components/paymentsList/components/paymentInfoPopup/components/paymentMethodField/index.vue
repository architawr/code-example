<template lang="pug">
li.fields-table__item
  .fields-table__label Метод платежа
  .fields-table__body
    .fields-table__fields
      .fields-table__field
        Let(
          :val="paymentMethod(payment.paymentMethod)"
          #default="{ val: paymentMethodReady }"
        )
          //- v-progress-linear(v-if="regionsIsLoading" :indeterminate="true").loader.loader--min-w.loader--full
          //- template(v-else)
          | {{ paymentMethodReady ? paymentMethodReady.name : payment.paymentMethod }}
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
      paymentMethod: 'blocks/payments/getMethod'
    })
  }
};
</script>
