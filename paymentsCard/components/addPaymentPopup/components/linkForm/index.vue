<template lang="pug">
v-form(
  ref="linkForm"

  v-model="valid"

  @submit.prevent="getLink"
).fields-table
  ul.fields-table__list
    authorization-type-field(
      v-model="linkInfo.authorizationTypeId"
    )

    paid-up-field(
      v-model="linkInfo.paidUp"

      @change="onPaidUpChange"
    )

    li.fields-table__item
      .fields-table__label Описание платежа
      .fields-table__body
        .fields-table__fields
          .fields-table__field.fields-table__field--full
            v-text-field(
              flat
              type="text"

              v-model="linkInfo.description"

              single-line
            )

    li.fields-table__item
      .fields-table__label Срок жизни ссылки
      .fields-table__body
        .fields-table__fields
          .fields-table__field
            v-text-field(
              flat
              type="number"
              min="1"

              v-model.number="sessionTimeoutRaw"

              suffix="мин"
              single-line
            )

    all-order-field(
      v-model="isLinkForAllOrder"

      :productsIsLoading="productsIsLoading"
      :servicesIsLoading="servicesIsLoading"

      @change="onIsLinkForAllOrderChange"
    )

    template(v-if="!isLinkForAllOrder")
      products-field(
        v-model="productsRaw"

        :products="products"
        :productsIsLoading="productsIsLoading"

        @change="onCartChange"
      )

      services-field(
        v-model="servicesRaw"

        :services="services"
        :servicesIsLoading="servicesIsLoading"

        @change="onCartChange"
      )

    template(
      v-if="link || linkIsLoading"
    )
      v-progress-linear(v-if="linkIsLoading" :indeterminate="true").loader
      li(v-if="link && !linkIsLoading").fields-table__item.payment-link__wrapper
        .fields-table__body.fields-table__body--full
          .fields-table__fields.fields-table__fields--between
            .fields-table__field
              a(
                :href="link"

                target="_blank"
              ).payment-link {{ link }}

            .fields-table__field.payment-link__actions
              v-btn(
                text

                color="accent2"

                @click="copyLink"
              ).payment-link__action
                v-icon content_copy
                | Скопировать

              v-btn(
                text

                :loading="orderIsLoading || linkIsSendingByEmail"

                @click="sendLinkByEmail"
              ).accent2.darken-3.white--text.payment-link__action Отправить по Email

              v-btn(
                text

                :loading="orderIsLoading || linkIsSendingBySms"

                @click="sendLinkBySms"
              ).accent2.darken-3.white--text.payment-link__action Отправить по СМС

</template>

<script>
import authorizationTypeField from './components/authorizationTypeField';
import paidUpField from './components/paidUpField';
import allOrderField from './components/allOrderField';
import productsField from './components/productsField';
import servicesField from './components/servicesField';

import { mapState } from 'vuex';

import { rules } from '@/utils/forms.js';

import cloneDeep from 'lodash/cloneDeep';
import isEqual from 'lodash/isEqual';

const getPhoneReady = phone => phone.replace(/\s|\+|\(|\)|-/g, '');

const EMPTY_PAYMENT = {
  authorizationTypeId: null,
  amount: null,
  paidUp: null,
  description: null,
  sessionTimeout: null,
  formPageLayout: 'desktop',
  cart: []
};

const getReadySessionTimeout = sessionTimeout => {
  const SECONDS_IN_MINUTE = 60;

  return sessionTimeout * SECONDS_IN_MINUTE;
};

export default {
  data: () => ({
    valid: false,

    isLinkForAllOrder: false,

    productsRaw: [],
    servicesRaw: [],

    linkInfo: cloneDeep(EMPTY_PAYMENT),
    sessionTimeoutRaw: null,

    cachedLinkInfoReady: null,

    link: null,
    linkIsLoading: false,

    linkIsSendingByEmail: false,
    linkIsSendingBySms: false,

    rules
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

  computed: {
    ...mapState({
      order: state => state.order.info,
      orderIsLoading: state => state.order.isLoading
    }),

    getLinkDisabled() {
      return isEqual(this.linkInfoReady, this.cachedLinkInfoReady);
    },

    linkInfoReady() {
      const { totalCost: orderTotalCost } = this.order;

      return {
        ...this.linkInfo,
        amount: orderTotalCost,

        sessionTimeout: getReadySessionTimeout(this.sessionTimeoutRaw),
        cart: this.cartReady
      };
    },

    cartReady() {
      const productsRaw = this.productsRaw.reduce((acc, product) => {
        const { productId, quantity, price, discount, tax } = product;

        acc.push({
          productId,
          quantity,
          price,
          discount,
          vatValue: quantity * (tax || 0)
        });

        return acc;
      }, []);

      const servicesRaw = this.servicesRaw.reduce((acc, service) => {
        const { serviceId, quantity, price, discount, tax } = service;

        acc.push({
          productId: serviceId,
          quantity,
          price,
          discount,
          vatValue: quantity * (tax || 0)
        });

        return acc;
      }, []);

      return [...productsRaw, ...servicesRaw];
    },
    cartReadyPrice() {
      return this.cartReady.reduce(
        (acc, cartItem) =>
          acc +
          ((cartItem.price || 0) - (cartItem.discount || 0)) * (cartItem.quantity || 1),
        0
      );
    }
  },

  methods: {
    async getLink() {
      this.$refs.linkForm.validate();

      if (!this.valid) return;

      await this.fetchLink();
    },

    async fetchLink() {
      const { clientOrderId } = this.order;

      this.linkIsLoading = true;

      try {
        const {
          data: { link }
        } = await this.$store.dispatch('payments/onlinepay/getLink', {
          clientOrderId,
          payload: this.linkInfoReady
        });

        this.cachedLinkInfoReady = cloneDeep(this.linkInfoReady);

        this.link = link;
      } catch {
        this.$toast.error('Не удалось сформировать ссылку на оплату.');
      } finally {
        this.linkIsLoading = false;
      }
    },

    copyLink() {
      const dummy = document.createElement('textarea');

      document.body.appendChild(dummy);

      dummy.value = this.link;

      dummy.select();
      dummy.setSelectionRange(0, 99999);

      document.execCommand('copy');

      document.body.removeChild(dummy);
    },

    async sendLinkByEmail() {
      this.linkIsSendingByEmail = true;

      try {
        const messageTypeId = await this.getDefaultEmailSendingService();

        const { id: orderId, contactEmail } = this.order;

        if (!contactEmail) {
          this.$toast.error('Необходимо заполнить поле Email.');

          return;
        }

        await this.$store.dispatch('message/sendMsg', {
          messageTypeId,
          payload: {
            templateId: 'PAYMENT_LINK_NOTIFY',
            orderId,
            templateData: {
              totalCost: this.paidUp,
              paymentLink: this.link
            },
            recipient: [contactEmail]
          }
        });

        this.$toast.success('Email успешно отправлено.');
      } catch {
        this.$toast.error('Не удалось отправить Email.');
      } finally {
        this.linkIsSendingByEmail = false;
      }
    },
    async getDefaultEmailSendingService() {
      const { data: defaultEmailSendingService } = await this.$store.cache.dispatch(
        'settings/getValue',
        'default_email_sending_service'
      );

      return defaultEmailSendingService;
    },

    async sendLinkBySms() {
      this.linkIsSendingBySms = true;

      try {
        const messageTypeId = await this.getDefaultSmsSendingService();

        const { id: orderId, contactPhone } = this.order;

        if (!contactPhone) {
          this.$toast.error('Необходимо заполнить поле Телефон.');

          return;
        }

        const contactPhoneReady = getPhoneReady(contactPhone);

        await this.$store.dispatch('message/sendMsg', {
          messageTypeId,
          payload: {
            templateId: 'PAYMENT_LINK_NOTIFY',
            orderId,
            templateData: {
              totalCost: this.paidUp,
              paymentLink: this.link
            },
            recipient: [contactPhoneReady]
          }
        });

        this.$toast.success('СМС успешно отправлено.');
      } catch {
        this.$toast.error('Не удалось отправить СМС.');
      } finally {
        this.linkIsSendingBySms = false;
      }
    },
    async getDefaultSmsSendingService() {
      const { data: defaultSmsSendingService } = await this.$store.cache.dispatch(
        'settings/getValue',
        'default_sms_sending_service'
      );

      return defaultSmsSendingService;
    },

    resetValidation() {
      this.$refs.linkForm.resetValidation();
    },

    onPaidUpChange() {
      this.isLinkForAllOrder = false;

      this.productsRaw = [];
      this.servicesRaw = [];
    },
    onIsLinkForAllOrderChange(isLinkForAllOrder) {
      if (isLinkForAllOrder) {
        const productsRaw = this.products.reduce((acc, product) => {
          const { productId, quantity, price, discount, tax } = product;

          const accProduct = acc.find(accProduct => accProduct.productId === productId);

          if (accProduct) {
            this.$set(accProduct, 'quantity', accProduct.quantity + quantity);
            this.$set(accProduct, 'vatValue', accProduct.quantity * (tax || 0));
          } else {
            acc.push({
              productId,
              quantity,
              price,
              discount,
              vatValue: quantity * (tax || 0)
            });
          }

          return acc;
        }, []);

        const servicesRaw = this.services.reduce((acc, service) => {
          const { serviceId, quantity, price, discount, tax } = service;

          acc.push({
            serviceId,
            quantity,
            price,
            discount,
            vatValue: quantity * (tax || 0)
          });

          return acc;
        }, []);

        this.productsRaw = productsRaw;
        this.servicesRaw = servicesRaw;

        this.linkInfo.paidUp = this.cartReadyPrice;
      } else {
        this.productsRaw = [];
        this.servicesRaw = [];

        this.linkInfo.paidUp = null;
      }
    },

    onCartChange() {
      this.linkInfo.paidUp = this.cartReadyPrice;
    }
  },

  watch: {
    getLinkDisabled: {
      handler(getLinkDisabled) {
        this.$emit('getLinkDisabledChanged', getLinkDisabled);
      },
      immediate: true
    }
  },

  components: {
    authorizationTypeField,
    paidUpField,
    allOrderField,
    productsField,
    servicesField
  }
};
</script>

<style lang="scss" scoped>
.payment-link {
  color: var(--v-accent2-darken1);
  text-decoration: none;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;

  &:hover,
  &:focus {
    color: var(--v-accent2-base);
  }

  &__wrapper {
    margin-top: 16px;
  }

  &__actions {
    margin-top: 8px;
  }

  &__action {
    margin: 0;

    &:not(:last-child) {
      margin-right: 8px;
    }
  }
}
</style>
