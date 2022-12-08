<template>
  <div class="text-center">
    <q-avatar class="q-mb-md" size="xl" square>
      <img :src="`/img/icon.png`" />
    </q-avatar>
    <div class="text-h4 q-mt-md text-primary">Order History</div>
    <div class="text-positive text-h6 q-mt-lg q-mb-lg">
      {{ state.status }}
    </div>
  </div>
  <q-list separator>
    <q-item>
      <q-item-section>
        <q-item-label class="q-ml-sm text-center text-h6 text-primary"
          >#</q-item-label
        >
      </q-item-section>
      <q-item-section class="q-ml-sm text-center text-h6 text-primary"
        >Date</q-item-section
      >
    </q-item>
    <q-item
      clickable
      v-for="order in state.orders"
      :key="order.id"
      @click="selectOrder(order.id)"
    >
      <q-item-section class="text-center text-bold text-secondary">
        {{ order.id }}
      </q-item-section>
      <q-item-section class="text-center text-bold text-secondary">
        {{ formatDate(order.orderDate) }}
      </q-item-section>
    </q-item>
  </q-list>
  <q-dialog
    v-model="state.selectedAnOrder"
    transition-show="rotate"
    transition-hide="rotate"
  >
    <q-card>
      <q-card-actions align="right">
        <q-btn flat label="X" color="primary" v-close-popup class="text-h5" />
      </q-card-actions>
      <div class="text-center">
        <q-avatar class="q-mb-md center" size="x3" square>
          <img :src="`/img/icon.png`" />
        </q-avatar>
      </div>
      <q-card-section>
        <div class="text-h5 text-center text-primary">
          Order: #{{ state.orderDetails[0].orderId }}
        </div>
        <div class="text-h8 text-center text-secondary">
          Made on: {{ state.orderDetails[0].dateCreated }}
        </div>
        <q-card class="q-ma-md">
          <q-list separator>
            <q-item
              ><q-item-section class="q-ml-sm text-left text-h9 text-primary"
                >Name
              </q-item-section>
              <q-item-section class="text-center q-ml-sm text-h9 text-primary"
                >Quantities
                <q-item-label>S O B</q-item-label>
              </q-item-section>
              <q-item-section class="text-right q-ml-sm text-h9 text-primary"
                >Extended</q-item-section
              ></q-item
            >
            <q-item v-for="detail in state.orderDetails" :key="detail.orderId">
              <q-item-section>
                {{ detail.productName }}
              </q-item-section>
              <q-item-section class="text-center">
                {{ detail.qtySold }}
                {{ detail.qtyOrdered }}
                {{ detail.qtyOnBackOrder }}
              </q-item-section>
              <q-item-section class="text-right">
                {{ formatCurrency(detail.qtyOrdered * detail.amount) }}
              </q-item-section>
            </q-item>
            <q-item>
              <q-item-section
                class="q-ml-sm text-h7 text-bold text-primary text-right"
                >Sub:</q-item-section
              >
              <q-item-section class="col-4 text-center">
                {{ formatCurrency(state.sub) }}
              </q-item-section>
            </q-item>
            <q-item>
              <q-item-section
                class="q-ml-sm text-h7 text-bold text-primary text-right padding-right"
                >Tax(13%):</q-item-section
              >
              <q-item-section class="col-4 text-center">
                {{ formatCurrency(state.tax) }}
              </q-item-section>
            </q-item>
            <q-item>
              <q-item-section
                class="q-ml-sm text-h6 text-secondary text-right padding-right"
                >Total:</q-item-section
              >
              <q-item-section class="col-4 text-center">
                {{ formatCurrency(state.total) }}
              </q-item-section>
            </q-item>
          </q-list>
        </q-card>
      </q-card-section>
      <q-card-section
        class="text-center text-positive text-h6 q-mb-xs q-mt-lg q-pa-none"
      >
        {{ state.dialogStatus }}
      </q-card-section>
    </q-card>
  </q-dialog>
</template>
<script>
import { formatCurrency } from "../utils/formatutils";
import { reactive, onMounted } from "vue";
import { fetcher } from "../utils/apiutil";
import { formatDate } from "../utils/formatutils";
export default {
  setup() {
    let state = reactive({
      status: "",
      sub: 0,
      tax: 0,
      total: 0,
      orders: [],
      dialogStatus: "",
      selectedAnOrder: false,
      orderDetails: [],
      orderSelected: {},
    });
    onMounted(() => {
      loadOrders();
    });
    const loadOrders = async () => {
      try {
        let customer = JSON.parse(sessionStorage.getItem("customer"));
        const payload = await fetcher(`order/${customer.email}`);
        if (payload.error === undefined) {
          state.orders = payload;
          state.status = `loaded ${state.orders.length} orders`;
        } else {
          state.status = payload.error;
        }
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };
    const selectOrder = async (orderId) => {
      try {
        let customer = JSON.parse(sessionStorage.getItem("customer"));
        const payload = await fetcher(`order/${orderId}/${customer.email}`);
        state.orderDetails = payload;
        state.dialogStatus = `details for order`;
        state.selectedAnOrder = true;
        state.orderSelected = state.orders.find(
          (order) => order.id === orderId
        );
        state.sub = 0;
        state.tax = 0;
        state.total = 0;
        state.orderDetails.forEach((detail) => {
          state.sub += detail.amount * detail.qtyOrdered;
          state.tax += detail.amount * detail.qtyOrdered * 0.13;
          state.total += detail.amount * detail.qtyOrdered * 1.13;
        });
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    return {
      state,
      formatDate,
      formatCurrency,
      selectOrder,
    };
  },
};
</script>
