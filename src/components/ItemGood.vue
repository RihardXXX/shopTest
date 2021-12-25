<template>
  <b-list-group-item>
    <div class="d-flex justify-content-between align-items-center">
      {{ nameGood }}
    </div>
    <hr>
    <div class="d-flex justify-content-between align-items-center">
      <b-badge variant="primary" pill>
        цена: {{ converter(price) }} ₽
      </b-badge>
      <b-badge variant="primary" pill>
        шт.: {{ total }}
      </b-badge>
      <b-button
          variant="outline-primary"
          @click="$emit('add-cart')"
      >
        добавить в корзину
      </b-button>
    </div>
  </b-list-group-item>
</template>

<script>
  export default {
    name: 'ItemGood',
    props: {
      id: {
        type: String,
        required: true
      },
      nameGood: {
        type: String,
        required: true
      },
      price: {
        type: [Number, String],
        required: true
      },
      total: {
        type: [Number, String],
        required: true
      },
      rateDollar: {
        type: Number,
        required: true
      }
    },
    // Фильтр для курса доллара
    methods: {
      converter(value) {
        if (!value) throw new Error('где деньги чудик')
        return value * this?.rateDollar
      }
    }
  }
</script>
