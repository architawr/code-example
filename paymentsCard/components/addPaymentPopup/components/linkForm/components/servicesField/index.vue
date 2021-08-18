<template lang="pug">
li(v-if="hasServices").fields-table__item
  .fields-table__label Примененные услуги
  .fields-table__body
    .fields-table__fields
      .fields-table__field
        v-select(
          flat
          multiple

          :items="services"
          :value="servicesRaw"
          :loading="servicesIsLoading"

          item-text="title"
          item-value="serviceId"

          single-line
          return-object

          @change="onChange"
        )
</template>

<script>
export default {
  model: {
    prop: 'servicesRaw',
    event: 'change'
  },

  props: {
    servicesRaw: {
      type: Array,
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
    hasServices() {
      return this.servicesIsLoading || (!this.servicesIsLoading && this.services?.length);
    }
  },

  methods: {
    onChange(servicesRaw) {
      this.$emit('change', servicesRaw);
    }
  }
};
</script>
