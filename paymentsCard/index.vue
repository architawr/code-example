<template lang="pug">
v-skeleton-loader(
  v-if="paymentsVisibility || paymentsVisibilityIsLoading"

  width="100%"
  :loading="paymentsVisibilityIsLoading"
  type="image"
).order-card__loader
  v-card(flat).order-card.order-card--transparent
    v-card-title.order-card__title Платежи

    .order-card__body.order-card__body--small-padding
      payments-list(
        :payments="payments"
        :paymentsIsLoading="paymentsIsLoading"

        :paymentsCanBeEdited="paymentsCanBeEdited"
      )

    v-card-actions.order-card__actions
      v-btn(
        v-if="addButtonVisible"

        text
        :ripple="false"

        :disabled="addButtonDisabled || paymentsIsLoading"

        @click="openAddPaymentPopup"
      ).order-card__action.order-card__action--small.order-card__action--no-mr-b + Добавить платеж

    add-payment-popup(
      ref="addPaymentPopup"

      :products="products"
      :productsIsLoading="productsIsLoading"

      :services="services"
      :servicesIsLoading="servicesIsLoading"

      @add="addPayment"
    )
</template>

<script>
import paymentsList from './components/paymentsList';
import addPaymentPopup from './components/addPaymentPopup';

import { mapState } from 'vuex';

export default {
	data: () => ({
		orderCalculationTimeout: null,
	}),

	props: {
		payments: {
			type: Array,
			required: true,
		},
		paymentsIsLoading: {
			type: Boolean,
			required: true,
		},

		products: {
			type: Array,
			required: true,
		},
		productsIsLoading: {
			type: Boolean,
			required: true,
		},

		services: {
			type: Array,
			default: () => [],
		},
		servicesIsLoading: {
			type: Boolean,
			default: false,
		},

		addButtonVisible: {
			type: Boolean,
			default: false,
		},
		addButtonDisabled: {
			type: Boolean,
			default: false,
		},

		paymentsCanBeEdited: {
			type: Boolean,
			default: false,
		},
	},

	created() {
		this.$store.cache.dispatch('blocks/payments/getVisibility');
	},

	computed: {
		...mapState({
			paymentsVisibility: (state) => state.blocks.payments.visibility,
			paymentsVisibilityIsLoading: (state) => state.blocks.payments.visibilityIsLoading,
		}),
	},

	methods: {
		addPayment(payment) {
			this.payments.push(payment);

			this.getOrderCalculation();

			this.closeAddPaymentPopup();
		},

		getOrderCalculation() {
			if (this.orderCalculationTimeout) clearTimeout(this.orderCalculationTimeout);

			this.orderCalculationTimeout = setTimeout(() => {
				const {
					info: order,
					items: { list: orderItems },
					services: { list: orderServices },
					shipping: { info: orderShipping },
					marketing: { info: orderMarketing },
				} = this.$store.state.order;

				const payload = {
					order,
					items: orderItems,
					itemServices: orderServices,
					shipping: orderShipping,
					marketing: orderMarketing,
				};

				this.$store.dispatch('order/calculation/getInfo', payload);
			}, 300);
		},

		openAddPaymentPopup() {
			this.$refs.addPaymentPopup.open();
		},
		closeAddPaymentPopup() {
			this.$refs.addPaymentPopup.close();
		},
	},

	components: {
		paymentsList,
		addPaymentPopup,
	},
};
</script>
