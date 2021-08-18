<template lang="pug">
li.fields-table__item
  .fields-table__label Тип списания
  .fields-table__body
    v-skeleton-loader(
      width="100%"
      :loading="isEditableIsLoading || typesIsLoading"
      type="heading"
    )
      div(v-if="isEditable || !defaultType").fields-table__fields
        .fields-table__field
          v-select(
            flat

            :items="types"
            :value="typeId"

            :loading="typesIsLoading"

            item-text="name"
            item-value="id"

            single-line

            :rules="[rules.required]"

            @change="onChange"
          )
      div(v-else).fields-table__fields
        .fields-table__field
          Let(
            :val="type(typeId)"
            #default="{ val: typeReady }"
          )
            v-progress-linear(v-if="typesIsLoading" :indeterminate="true").loader.loader--min-w.loader--full
            template(v-else) {{ typeReady ? typeReady.name : typeId  }}
</template>

<script>
import { mapState, mapGetters } from 'vuex';

import { rules } from '@/utils/forms.js';

export default {
  data: () => ({
    rules
  }),

  model: {
    prop: 'typeId',
    event: 'change'
  },

  props: {
    typeId: {
      type: String,
      default: null
    }
  },

  created() {
    this.initialFetch();
  },

  computed: {
    ...mapState({
      isEditable: state => state.blocks.payments.fields.authorizationType.isEditable,
      isEditableIsLoading: state =>
        state.blocks.payments.fields.authorizationType.isEditableIsLoading,

      types: state => state.blocks.payments.fields.authorizationType.list,
      typesIsLoading: state =>
        state.blocks.payments.fields.authorizationType.listIsLoading
    }),

    ...mapGetters({
      type: 'blocks/payments/fields/authorizationType/getAuthorizationType'
    }),

    defaultType() {
      return this.types?.find(type => type.defaultType);
    }
  },

  methods: {
    async initialFetch() {
      await this.$store.cache.dispatch(
        'blocks/payments/fields/authorizationType/getInfo'
      );

      if (this.typeId) return;

      this.setDefaultType();
    },
    setDefaultType() {
      if (!this.defaultType) return;

      this.onChange(this.defaultType.id);
    },

    onChange(typeId) {
      this.$emit('change', typeId);
    }
  }
};
</script>
