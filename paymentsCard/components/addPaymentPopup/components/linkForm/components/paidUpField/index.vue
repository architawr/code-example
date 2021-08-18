<template lang="pug">
li.fields-table__item
  .fields-table__label Сумма
  .fields-table__body
    v-skeleton-loader(
      width="100%"
      :loading="isEditableIsLoading"
      type="heading"
    )
      div(v-if="isEditable").fields-table__fields
        .fields-table__field
          v-text-field(
            flat
            type="number"
            min="1"

            :value="value"

            :suffix="orderCurrencyLabel"
            single-line

            :rules="[rules.required]"

            @change="onChange"
          )
      div(v-else).fields-table__fields
        .fields-table__field {{ value }} {{ orderCurrencyLabel }}
</template>

<script>
import { mapState } from 'vuex';

import { rules } from '@/utils/forms.js';

export default {
  data: () => ({
    rules
  }),

  model: {
    prop: 'value',
    event: 'change'
  },

  props: {
    value: {
      type: Number,
      default: 0
    }
  },

  created() {
    this.initialFetch();
  },

  computed: {
    ...mapState({
      order: state => state.order.info,

      isEditable: state => state.blocks.payments.fields.sumPaid.isEditable,
      isEditableIsLoading: state =>
        state.blocks.payments.fields.sumPaid.isEditableIsLoading
    }),

    orderCurrencyLabel() {
      const { uppercase } = this.$options.filters;

      return this.$t(
        uppercase('SETTINGS.CURRENCIES.' + this.order.currencyId + '.SHORT')
      );
    }
  },

  methods: {
    async initialFetch() {
      await this.$store.cache.dispatch('blocks/payments/fields/sumPaid/getInfo');
    },

    onChange(value) {
      this.$emit('change', +value);
    }
  }
};
</script>
