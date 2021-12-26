<template>
  <b-container fluid>
    <template v-if="!isLoading">
      <b-modal ref="my-modal">
        Данный товар уже закончился на складе (((
      </b-modal>
      <b-row>
        <b-col>
          <div class="mt-2">
            <b-badge :variant="colorBadge">
              курс валют: {{ rateDollar }}
            </b-badge>
            <b-form-input
                id="range-2"
                v-model="rateDollar"
                type="range"
                min="20"
                max="80"
                step="1">
            </b-form-input>
          </div>
        </b-col>
      </b-row>
      <b-row>
        <b-col
            v-if="groupsList.length"
            class="section"
        >
          <Group
              v-for="(group, index) in groupsList"
              :key="group"
              :groupName="group"
              :accordionName="`accordion${index+1}`"
          >
            <template v-if="checkProductGroup(group, goodsBase)">
              <ItemGood
                  v-for="product in sortProductGroup(group)"
                  :key="product.id"
                  :id="product.id"
                  :name-good="product.product.N"
                  :total="product.total"
                  :rate-dollar="Number(rateDollar)"
                  :price="product.priceDollar"
                  @add-cart="() => addCart(product.id)"
              />
            </template>
            <template v-else>
              <b-alert show variant="danger">
                к сожалению товары данной категории отсутствуют
              </b-alert>
            </template>
          </Group>
        </b-col>
        <b-col>
          <Group
              groupName="Корзина"
              :accordionName="`accordion${12121212121}`"
          >
            <template v-if="cart.length">
              <ItemCart
                  v-for="product in cart"
                  :key="product.id"
                  :id="product.id"
                  :name-good="product.product.N"
                  :rate-dollar="Number(rateDollar)"
                  :price="product.priceDollar"
                  :qty="Number(product.qty)"
                  :total="product.total"
                  @delete-cart="deleteCart(product.id)"
                  @decrementCart="decrementCart"
              />
            </template>
            <template v-else>
              <b-alert show variant="danger">
                товары в корзине отсутствуют
              </b-alert>
            </template>
          </Group>
          <b-alert show variant="primary">
            Товаров в корзине на сумму: {{ cartSum }} ₽
          </b-alert>
        </b-col>
      </b-row>
    </template>
      <div v-else
           :style="{marginTop: '5px'}"
      >
        <flip-countdown
            :deadline="deadLine"
            :showDays="false"
        ></flip-countdown>
        <div :style="{textAlign: 'center'}">
          <b-button variant="primary" disabled>
            <b-spinner big></b-spinner>
            <span class="sr-only">Loading...</span>
          </b-button>
        </div>
      </div>
  </b-container>
</template>

<script>
  import FlipCountdown from 'vue2-flip-countdown'
  import data from '@/server/data.json'
  import names from '@/server/names.json'
  import Group from '@/components/Group'
  import ItemGood from "../components/ItemGood";
  import { v4 as uuidv4 } from 'uuid'
  import ItemCart from "../components/ItemCart";
  import dateFormat from "dateformat";
  const now = new Date();
  now.setSeconds(now.getSeconds() + 15);

  export default {
    name: 'Home',
    data() {
      return {
        // data: data.Value.Goods,
        names: names,
        // данные с сервера с товарами для парсинга
        goodsBase: [],
        // данные с сервера с группами
        groupsList: [],
        // курс валюты
        rateDollar: 65,
        // корзина
        cart: [],
        // состояние прогрузки из сервера
        isLoading: true,
        // дедлайн для таймера
        deadLine: null,
      }
    },
    components: {
      ItemCart,
      ItemGood,
      Group,
      FlipCountdown
    },
    computed: {
      // Сумма товаров в корзине
      cartSum() {
        return this.cart.reduce((acc, current) => {
          return acc + (current.priceDollar * current.qty * this.rateDollar)
        }, 0)
      },
      colorBadge() {
        return this.rateDollar <= 50 ? 'success' : 'danger'
      }
    },
    methods: {
      // ====================Блок управления магазином===========================
      // сортировка товаров по группам
      // Ресурсозатратно для шаблона, но структура данных такая
      sortProductGroup(groupName) {
        return this.goodsBase.filter(product => product.group === groupName)
      },
      // проверка на вхождение группы товара чтобы сообщать юзеру об их отсутствии
      checkProductGroup(group, gooodList) {
        return gooodList.some(product => product.group === group)
      },
      addCart(id) {
        // находим товар из склада
        const product = this.goodsBase.find(good => good.id === id)

        // кладём только если не 0
        if (product.total === 0) {
          // console.log('товара 0 выходим из добавления и ввызываем модалку')
          this.$refs['my-modal'].show()
          return
        }

        // если товар ранее добавлен то увеличиваем его количество в корзине
        const repeat = this.cart.some(product => product.id === id)
        if (repeat) {
          // console.log('добавляем тот же товар')
          this.cart = this.cart.map(product => {
            if (product.id === id) {
              product.qty += 1
              return product
            }
            return product
          })

          this.actualizationGoodsInStock(id)
          return
        }

        // Кладём товар в корзину
        this.cart.push({
          ...product,
          qty: 1
        })

        // уменьшаем на складе товар
        this.actualizationGoodsInStock(id)
      },
      // Уменьшение товара на складе при добавлении товара в корзину
      actualizationGoodsInStock(id) {
        this.goodsBase = this.goodsBase.map(good => {
          if (good.id === id) {
            // если товара 0 то не даем добавлять товар в корзину
            if (good.total === 0) {
              return
            }
            good.total = good.total -= 1
            return good
          }
          return good
        })
      },
      // актуализаци товара на складе после удаления товара с корзины полностью
      actualizationGoodsInStockDelProd(product) {
        // console.log('посчитать количество товара которое было в корзине ', product)
        const addTotal = product.qty
        this.goodsBase = this.goodsBase.map(good => {
          if (good.id === product.id) {
            good.total += addTotal
            return good
          }
          return  good
        })
      },
      // удаление товара из корзины полностью
      deleteCart(id) {
        // console.log('delete product', id)
        const product = this.cart.find(good => good.id === id)
        this.cart = this.cart.filter(product => product.id !== id)
        this.actualizationGoodsInStockDelProd(product)
      },

      // ======================Блок управления корзиной=============================

      // Уменьшение товаров с корзины согласно id товара и количества
      decrementCart(id, count, total) {
        // на склад добавить разницу между количеством и корзиной то есть чтобы на складе было актуальное
        const totalGoods = total - count
        this.goodsBase = this.goodsBase.map(product => {
          if(product.id === id) {
            product.total = totalGoods
            return product
          }
          return product
        })

        // в корзину добавим актуальное количество для перерендринга
        this.cart = this.cart.map(product => {
          if(product.id === id) {
            product.qty = count
            return product
          }
          return product
        })

      },

      // Инициализация состояния данных из JSON
      initialState() {
        this.isLoading = false
        // Группы товаров
        this.groupsList = Object.values(names).map(item => item.G)

        // Распарсенные товары всё согласно двум файлам
        this.goodsBase = data.Value.Goods.map(item =>
            ({
              id: uuidv4(),
              group: names[item.G].G,
              product: names[item.G].B[item.T],
              priceDollar: Math.round(item.C),
              total: item.P
            }))
      }
    },

    mounted() {
      setTimeout(() => {
        this.initialState()
      }, 15000)
    },

    beforeMount() {
      // Эта шляпа нужна для дедлайна таймера
      //2021-12-25 21:00:00
      const currentDate = dateFormat(now, 'yyyy-mm-dd')
      const currentTime = dateFormat(now,'HH:MM:ss')
      this.deadLine = `${currentDate} ${currentTime}`

    }
  }
</script>
