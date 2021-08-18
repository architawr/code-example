<template lang="pug">
v-text-field(
  type="number"
  :max="maxValue"

  :value="value"
  :loading="isEditableIsLoading"
  :disabled="!isEditable"

  name="price"
  label="Цена"
  height="20"
  :suffix="orderCurrencyLabel"

  :rules="[rules.required, rules.onlyPositiveNumbers, maxValueRule]"

  @change="onChange"
)
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
    },

    maxValue: {
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

      isEditable: state => state.blocks.payments.fields.cart.isEditable,
      isEditableIsLoading: state => state.blocks.payments.fields.cart.isEditableIsLoading
    }),

    maxValueRule() {
      if (!this.maxValue) return v => !!v;

      return v =>
        (v || 0) <= this.maxValue || `Цена не должна превышать реальную цену товара.`;
    },

    orderCurrencyLabel() {
      const { uppercase } = this.$options.filters;

      return this.$t(
        uppercase('SETTINGS.CURRENCIES.' + this.order.currencyId + '.SHORT')
      );
    }
  },

  methods: {
    async initialFetch() {
      await this.$store.cache.dispatch('blocks/payments/fields/cart/getInfo');
    },

    onChange(value) {
      this.$emit('change', +value);
    }
  }
};
</script>
