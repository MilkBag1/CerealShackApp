<template>
  <div class="text-center">
    <div class="text-h4 q-mt-md">Cart Contents</div>
    <q-avatar class="q-mt-md" size="xl" square>
      <img :src="`/img/cart.png`" />
    </q-avatar>
    <div class="text-h6 text-positive">{{ state.status }}</div>
  </div>
  <div
    v-if="state.cart.length > 0"
    style="width: 90vw; margin-left: 5vw; margin-top: 1vh"
  ></div>
  <div>
    <q-item style="bottom: -2vh">
      <q-item-section class="col-2 q-ml-sm text-h6 text-primary"
        >Name</q-item-section
      >
      <q-item-section class="q-ml-sm text-h6 text-primary text-center"
        >Qty</q-item-section
      >
      <q-item-section class="q-ml-sm text-h6 text-primary text-center"
        >MSRP</q-item-section
      >
      <q-item-section class="q-ml-sm text-h6 text-primary"
        >Extended</q-item-section
      >
    </q-item>
    <q-scroll-area style="height: 40vh">
      <q-card class="q-ma-md">
        <q-list separator>
          <q-item v-for="item in state.cart" :key="item.id" clickable>
            <q-item-section class="col-4">
              {{ item.item.productName }}
            </q-item-section>
            <q-item-section class="text-left">
              {{ item.qty }}
            </q-item-section>
            <q-item-section class="text-left">
              {{ formatCurrency(item.item.msrp) }}
            </q-item-section>
            <q-item-section class="text-left">
              {{ formatCurrency(item.item.msrp * item.qty) }}
            </q-item-section>
          </q-item>
          <q-item>
            <q-item-section class="col-4"></q-item-section>
            <q-item-section class="text-left"></q-item-section>
            <q-item-section class="q-ml-sm text-h6 text-primary text-center"
              >Sub:</q-item-section
            >
            <q-item-section class="col-4 text-center">
              {{ formatCurrency(state.sub) }}
            </q-item-section>
          </q-item>
          <q-item>
            <q-item-section class="col-4"></q-item-section>
            <q-item-section class="text-left"></q-item-section>
            <q-item-section
              class="q-ml-sm text-h7 text-bold text-primary text-left padding-right"
              >Tax(13%):</q-item-section
            >
            <q-item-section class="col-4 text-center">
              {{ formatCurrency(state.tax) }}
            </q-item-section>
          </q-item>
          <q-item>
            <q-item-section class="col-4"></q-item-section>
            <q-item-section class="text-left"></q-item-section>
            <q-item-section
              class="q-ml-sm text-h6 text-secondary text-left padding-right"
              >Total:</q-item-section
            >
            <q-item-section class="col-4 text-center">
              {{ formatCurrency(state.total) }}
            </q-item-section>
          </q-item>
        </q-list>
      </q-card>
    </q-scroll-area>
    <div class="col-4 q-mr-sm text-center">
      <q-btn
        class="q-mr-sm"
        color="primary"
        label="Checkout"
        :disable="state.cart.length < 1"
        @click="checkout()"
      />
      <q-btn
        color="primary"
        label="Clear Cart"
        :disable="state.cart.length < 1"
        @click="clearCart()"
      />
    </div>
  </div>
</template>
<script>
import { formatCurrency } from "../utils/formatutils";
import { poster } from "../utils/apiutil";
import { reactive, onMounted } from "vue";
export default {
  setup() {
    onMounted(() => {
      if (sessionStorage.getItem("cart")) {
        state.cart = JSON.parse(sessionStorage.getItem("cart"));
        state.cart.forEach((cartItem) => {
          state.sub += cartItem.item.msrp * cartItem.qty;
          state.tax += cartItem.item.msrp * cartItem.qty * 0.13;
          state.total += cartItem.item.msrp * cartItem.qty * 1.13;
        });
      } else {
        state.cart = [];
      }
    });

    let state = reactive({
      sub: 0,
      tax: 0,
      total: 0,
      status: "",
      cart: [],
    });

    const clearCart = () => {
      sessionStorage.removeItem("cart");
      state.cart = [];
      state.status = "cart cleared";
      state.sub = 0;
      state.tax = 0;
      state.total = 0;
    };

    const checkout = async () => {
      let customer = JSON.parse(sessionStorage.getItem("customer"));
      let cart = JSON.parse(sessionStorage.getItem("cart"));
      try {
        state.status = "sending cart info to server";
        let orderHelper = { email: customer.email, selections: cart };
        let payload = await poster("Order", orderHelper);
        if (payload.indexOf("not") > 0) {
          state.status = payload;
        } else {
          clearCart();
          state.status = payload;
        }
      } catch (err) {
        console.log(err);
        state.status = `Error add cart: ${err}`;
      }
    };

    return {
      state,
      formatCurrency,
      checkout,
      clearCart,
    };
  },
};
</script>
