<template lang="pug">
.payments-list
  s-main-table(
    fluid

    without-header
    without-background

    :rows="payments"
    :columns="columns"

    :loading="paymentsIsLoading"

    emptyCellClass="main-table__empty-cell--no-padding"
  )
    template(#timePaidCellInner="{ cellValue: timePaid }")
      span.secondary--text.text--darken-2 {{ timePaid | formatDate('dd MMM, HH:mm') }}
    template(#sumPaidCellInner="{ cellValue: sumPaid, row: { transactionType } }")
      span(:class="sumPaid < 0 ? 'error--text' : ''").font-weight-medium.mr-1 {{ sumPaid }} {{ orderCurrencyLabel }}
      template(v-if="transactionType")
        v-icon(v-if="transactionType === 'debit'" color="success") trending_up
        v-icon(v-else-if="transactionType === 'credit'" color="error") trending_down
    template(#paymentMethodCellInner="{ cellValue: paymentMethodRaw, row: { acquierId } }")
      template(v-if="paymentMethodRaw === 'online' || paymentMethodRaw === 'acquiring'") {{ acquierId }}
      Let(
        v-if="(paymentMethodRaw !== 'online' && paymentMethodRaw !== 'acquiring') || ((paymentMethodRaw === 'online' || paymentMethodRaw === 'acquiring') && !acquierId)"

        :val="paymentMethod(paymentMethodRaw)"
        #default="{ val: paymentMethodReady }"
      ) {{ paymentMethodReady ? paymentMethodReady.name : paymentMethodRaw  }}

    template(#actionsCellInner="{ row: payment }")
      v-icon(
        size="20"

        @click="openPaymentInfoPopup(payment)"
      ).promo__action info

    template(#emptyState)
      span.payments__empty Платежей нет

  payment-info-popup(
    ref="paymentInfoPopup"
  )
</template>

<script>
import paymentInfoPopup from './components/paymentInfoPopup';

import SMainTable from '@/components/UI/SMainTable';

import { mapState, mapGetters } from 'vuex';

export default {
	data: () => ({
		columns: [
			{
				field: 'timePaid',
				tdClass: 'main-table__cell--small-padding',
			},
			{
				field: 'sumPaid',
				tdClass: 'main-table__cell--small-padding',
			},
			{
				field: 'paymentMethod',
				tdClass: 'main-table__cell--small-padding',
			},
			{
				field: 'actions',
				tdClass: 'main-table__cell--small-padding main-table__cell--little',
			},
		],
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

		paymentsCanBeEdited: {
			type: Boolean,
			required: true,
		},
	},

	computed: {
		...mapState({
			order: (state) => state.order.info,
		}),

		...mapGetters({
			paymentMethod: 'blocks/payments/getMethod',
		}),

		paymentsNotEmpty() {
			return !!this.payments?.length;
		},

		orderCurrencyLabel() {
			const { uppercase } = this.$options.filters;

			return this.$t(
				uppercase('SETTINGS.CURRENCIES.' + this.order.currencyId + '.SHORT')
			);
		},
	},

	methods: {
		openPaymentInfoPopup(payment) {
			this.$refs.paymentInfoPopup.open(payment);
		},
	},

	components: {
		paymentInfoPopup,

		SMainTable,
	},
};
</script>

<style lang="scss" scoped>
.payments {
	&__empty {
		color: var(--v-secondary-darken2);
	}
}
</style>
