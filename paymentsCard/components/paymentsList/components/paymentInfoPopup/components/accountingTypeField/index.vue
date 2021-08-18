<template lang="pug">
li.fields-table__item
  .fields-table__label Тип учета
  .fields-table__body
    .fields-table__fields
      .fields-table__field
        Let(
          :val="accountingType(payment.accountingType)"
          #default="{ val: accountingTypeReady }"
        )
          //- v-progress-linear(v-if="regionsIsLoading" :indeterminate="true").loader.loader--min-w.loader--full
          //- template(v-else)
          | {{ accountingTypeReady ? accountingTypeReady.name : payment.accountingType  }}
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
      accountingType: 'blocks/payments/getAccountingType'
    })
  }
};
</script>
