<template>
  <div class="col-sm-9">
    <div v-if="me"
         class="my-orders col-sm-12">
      <div v-if="me.order"
           class="my-orders-order-content">
        <div class="row">
          <div class="col-sm-12">
            <div class="my-orders-title hidden-xs">
              <h4>{{ $t('myOrders')}}</h4>
            </div>
          </div>
        </div>
        <div class="your-order-details-wrapper">
          <div class="your-order-details-title">
            <h5>{{ $t('title')}}</h5>
          </div>
          <div class="your-order-details">
            <span class="your-order-details-titles">
              {{ $t('orderNumber') }}
              <br>
              {{ $t('date') }}
            </span>
            <span class="your-order-details-info">
              <span data-test="details-order-number">{{ me.order.orderNumber }}</span>
              <br>
              <BaseDate :date="me.order.createdAt"
                        :format="'short'"
                        data-test="details-order-date"/>
            </span>
          </div>
          <button onclick="window.print()"
                  class="my-orders-print-receipt-btn hidden-xs">
            {{ $t('print') }}
          </button>
        </div>
        <ShipmentInfo />
        <PriceCalculation :cartLike="me.order"
                          class="my-orders-order-price-summary-wrapper"/>
        <div class="order-list-summary-titles-wrapper">
          <div class="row">
            <div class="col-sm-6">
              <div>{{ $t('description') }}</div>
            </div>
            <div class="col-sm-2">
              <div class="text-right">{{ $t('quantity') }}</div>
            </div>
            <div class="col-sm-2">
              <div class="text-right">{{ $t('price') }}</div>
            </div>
            <div class="col-sm-2">
              <div class="text-right">{{ $t('total') }}</div>
            </div>
          </div>
        </div>
        <LineItem v-for="lineItem in me.order.lineItems"
              :key="lineItem.id"
              :lineItem="lineItem"
              class="order-detail-wrapper"
              data-test="order-line-items"/>
      </div>
      <div v-else>
        <h1 class="text-center">{{ $t('notFound') }}</h1>
      </div>
    </div>
  </div>
</template>

<script>
import gql from 'graphql-tag';
import ShipmentInfo from './ShipmentInfo.vue';
import BaseDate from '../../common/BaseDate.vue';
import PriceCalculation from '../../common/PriceCalculation.vue';
import LineItem from '../../common/LineItem.vue';
import DisplayableMoneyFragment from '@/components/DisplayableMoney.gql';

export default {
  components: {
    ShipmentInfo,
    BaseDate,
    PriceCalculation,
    LineItem,
  },

  data: () => ({
    me: null,
  }),

  computed: {
    orderNumber() {
      return this.$route.params.orderNumber;
    },
  },

  apollo: {
    me: {
      query: gql`
        query orderByOrderNumber($orderNumber: String, $locale: Locale!) {
          me {
            order(orderNumber: $orderNumber) {
              id
              version
              orderNumber
              createdAt
              shippingInfo {
                price {
                  ...DisplayableMoney
                }
              }
              taxedPrice {
                totalGross {
                  ...DisplayableMoney
                }
                totalNet {
                  ...DisplayableMoney
                }
                taxPortions {
                  amount {
                    ...DisplayableMoney
                  }
                }
              }
              totalPrice {
                ...DisplayableMoney
              }
              lineItems {
                id
                quantity
                name(locale: $locale)
                productSlug(locale: $locale)
                price{
                    value {
                      ...DisplayableMoney
                    }
                    discounted {
                      value {
                        ...DisplayableMoney
                      }
                    }
                  }
                totalPrice {
                  ...DisplayableMoney
                }
                variant {
                  images {
                    url
                    label
                  }
                  sku
                }
              }
            }
          }
        }
        ${DisplayableMoneyFragment}`,
      variables() {
        return {
          orderNumber: this.orderNumber,
          locale: this.$i18n.locale,
        };
      },
    },
  },
};
</script>

<i18n>
en:
  title: "Your order details"
  myOrders: "My Orders"
  orderNumber: "Order Number"
  date: "Date"
  print: "Print receipt"
  description: "Description"
  quantity: "Quantity"
  price: "Price"
  total: "Total"
  notFound: "Order not found"
de:
  title: "Ihre Bestelldetails"
  myOrders: "Meine Bestellungen"
  orderNumber: "Bestellnummer"
  date: "Datum"
  print: "Beleg drucken"
  description: "Beschreibung"
  quantity: "Menge"
  price: "Preis"
  total: "Gesamtpreis"
  notFound: "Bestellung nicht gefunden"
</i18n>
