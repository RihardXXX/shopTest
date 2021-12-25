<template>
  <b-list-group-item :key="componentKey">
    <b-modal ref="my-modal">
      такого количества товара нет в магазине сорян (((
    </b-modal>
    <div class="d-flex justify-content-between align-items-center">
      {{ nameGood }}
    </div>
    <hr>
    <div class="d-flex justify-content-between align-items-center">
<!--      <b-badge variant="primary" pill>-->
<!--        количество: {{ qty }} шт.-->
<!--      </b-badge>-->
      <b-badge variant="primary" pill>
        цена: {{ converter(price) }} ₽ / 1 шт.
      </b-badge>
      <b-badge variant="primary" pill>
        цена: {{ converter(price) * counter }} ₽ / общая.
      </b-badge>
    </div>
    <hr>
    <div class="d-flex justify-content-between align-items-center">
      <b-button
          variant="outline-primary"
          @click="$emit('delete-cart')"
      >
        удалить из корзины
      </b-button>
      <label for="count">количество</label>
      <b-form-input
          id="count"
          type="number"
          :value="counter"
          :style="{ width: '70px' }"
          @input="(e) => this.counter = e"
      ></b-form-input>
    </div>
  </b-list-group-item>
</template>

<script>
  // import { debounce } from 'vue-debounce'

  export default {
    name: 'ItemCart',
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
      rateDollar: {
        type: Number,
        required: true
      },
      qty: {
        type: Number,
        required: true
      },
      total: {
        type: [Number, String],
        required: true
      }
    },
    data() {
      return {
        // для принудительного перерендера
        componentKey: 0,
      };
    },
    computed: {
      counter: {
        get() {
          return String(this.qty)
        },
        set(newQty) {
          console.log('newQty: ', newQty)
          if(newQty <= 0) {
            this.$emit('delete-cart', this.id)
          } else if(newQty > this.total) {
            this.$refs['my-modal'].show()
            // если количество заказываемых товаров превышает чем количество товаров в магазе
            setTimeout(() => this.componentKey += 1, 1000)
          } else {
            console.log('внесение изменений')
            // тут эмитим увеличение или уменьшение товара
            console.log(newQty)
            this.$emit('decrementCart', this.id, newQty, this.total)
          }
        }
      }
    },
    // Фильтр для курса доллара
    methods: {
      converter(value) {
        if (!value) throw new Error('где деньги чудик')
        return value * this?.rateDollar
      },
      // debounce(e) {
      //   return debounce(() => this.counter = e, 300)
      // }
    }
  }
</script>
