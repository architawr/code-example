<template lang="pug">
sweet-modal(
  ref="addPaymentPopup"

  width="720px"
  title="Добавить платеж"

  @close="onClose"
)
  template(v-if="tabsVisible")
    v-tabs(
      v-model="formType"

      color="accent2"
      grow

      @change="onFormTypeChange"
    )
      v-tab(
        key="link"
      ) Ссылка на оплату
      v-tab(
        key="default"
      ) Факт оплаты

    v-tabs-items(v-model="formType")
      v-tab-item(key="link")
        .add-payment-form__wrapper
          link-form(
            ref="linkForm"

            :products="products"
            :productsIsLoading="productsIsLoading"

            :services="services"
            :servicesIsLoading="servicesIsLoading"

            @getLinkDisabledChanged="updateTriggerGetLinkDisabled"
          ).add-payment-form
      v-tab-item(key="default")
        .add-payment-form__wrapper
          common-form(
            ref="commonForm"

            :payment="payment"

            :products="products"
            :productsIsLoading="productsIsLoading"

            :services="services"
            :servicesIsLoading="servicesIsLoading"
          ).add-payment-form

    v-tabs-items(slot="button" v-model="formType").v-tabs-items--transparent
      v-tab-item(key="link")
        v-btn(
          text

          :loading="orderIsLoading || orderItemServicesIsLoading || orderShippingIsLoading"
          :disabled="triggerGetLinkDisabled"

          @click.prevent="triggerGetLink"
        ).accent.dark--text Сформировать ссылку на оплату

      v-tab-item(key="default")
        v-btn(
          text

          @click.prevent="addPayment"
        ).accent.dark--text Добавить

  template(v-else)
    common-form(
      ref="commonForm"

      :payment="payment"

      :products="products"
      :productsIsLoading="productsIsLoading"

      :services="services"
      :servicesIsLoading="servicesIsLoading"
    ).add-payment-form

    v-btn(
      text

      slot="button"

      @click.prevent="addPayment"
    ).accent.dark--text Добавить
</template>

<script>
/* eslint-disable no-unused-expressions */

import commonForm from './components/commonForm';
import linkForm from './components/linkForm';

import { mapState } from 'vuex';

import cloneDeep from 'lodash/cloneDeep';

const DEFAULT_FORM_TYPE = 0;

const getReadyPayment = payment => ({
  ...payment,

  timePaid: new Date()
});

const EMPTY_PAYMENT = {
  paymentId: '',
  orderId: null,
  claimId: null,
  sumPaid: null,
  timePaid: null,
  paymentMethod: null,
  acquierId: null,
  acquierTransactionId: null,
  acquierTransactionStatusId: null,
  productIds: [],
  userId: null
};

export default {
  data: () => ({
    tabsVisible: false,
    formType: DEFAULT_FORM_TYPE,

    payment: cloneDeep(EMPTY_PAYMENT),

    triggerGetLinkDisabled: false
  }),

  props: {
    products: {
      type: Array,
      required: true
    },
    productsIsLoading: {
      type: Boolean,
      required: true
    },

    services: {
      type: Array,
      required: true
    },
    servicesIsLoading: {
      type: Boolean,
      required: true
    }
  },

  created() {
    this.fetchDefaultAcquier();
  },

  computed: {
    ...mapState({
      orderIsLoading: state => state.order.isLoading,
      orderItemServicesIsLoading: state => state.order.services.isLoading,
      orderShippingIsLoading: state => state.order.shipping.isLoading
    }),

    formTypeReady() {
      return `${this.formType || 'default'}Form`;
    }
  },

  methods: {
    async fetchDefaultAcquier() {
      const { data: defaultAcquier } = await this.$store.cache.dispatch(
        'settings/getValue',
        'default_acquier'
      );

      this.tabsVisible = !!defaultAcquier;
    },

    addPayment() {
      if (!this.$refs.commonForm?.validate()) return;

      this.$emit('add', getReadyPayment(this.payment));
    },
    triggerGetLink() {
      this.$refs.linkForm?.getLink();
    },
    updateTriggerGetLinkDisabled(triggerGetLinkDisabled) {
      this.triggerGetLinkDisabled = triggerGetLinkDisabled;
    },

    open() {
      this.$refs.addPaymentPopup.open();
    },
    close() {
      this.$refs.addPaymentPopup.close();
    },

    onClose() {
      this.clearPayment();

      this.resetFormsValidation();
    },

    clearPayment() {
      this.payment = cloneDeep(EMPTY_PAYMENT);
    },

    resetFormsValidation() {
      this.$refs.commonForm?.resetValidation();
      this.$refs.linkForm?.resetValidation();
    },

    onFormTypeChange() {
      this.resetFormsValidation();
    }
  },

  components: {
    commonForm,
    linkForm
  }
};
</script>

<style lang="scss" scoped>
.add-payment-form {
  &__wrapper {
    padding: 16px 0 0;

    &--no-pd {
      padding: 0;
    }
  }
}

.v-tabs-items--transparent {
  background-color: transparent !important;
}
</style>
