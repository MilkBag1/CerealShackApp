<template>
  <q-page>
    <div class="text-center q-mt-lg">
      <div class="text-h3">Our Brands</div>
      <div>
        <q-avatar class="q-mb-md" size="xl" square>
          <img :src="`/img/icon.png`" />
        </q-avatar>
      </div>
      <div class="text-positive q-mt-lg">{{ state.status }}</div>
      <q-select
        class="q-mt-lg q-ml-lg"
        v-if="state.categories.length > 0"
        style="width: 50vw; margin-bottom: 4vh; background-color: #fff"
        :option-value="'id'"
        :option-label="'name'"
        v-model="state.selectedCategoryId"
        :options="state.categories"
        label="Select a Category"
        emit-value
        map-options
        @update:model-value="loadMenuitems()"
      />
      <div
        class="text-h6 text-bold text-center text-primary"
        v-if="state.menuitems.length > 0"
      >
        {{ state.selectedCategory.name }} ITEMS
      </div>
      <q-scroll-area style="height: 55vh">
        <q-card class="q-ma-md">
          <q-list separator>
            <q-item
              v-for="item in state.menuitems"
              :key="item.id"
              clickable
              @click="selectMenuItem(item.id)"
            >
              <q-item-section avatar>
                <q-avatar style="height: 125px; width: 90px" square>
                  <img :src="`/img/${item.graphicName}`" />
                </q-avatar>
              </q-item-section>
              <q-item-section
                class="text-h7 text-bold text-center text-primary"
              >
                {{ item.productName }}
              </q-item-section>
            </q-item>
          </q-list>
        </q-card>
      </q-scroll-area>
    </div>
    <q-dialog
      v-model="state.itemSelected"
      transition-show="rotate"
      transition-hide="rotate"
    >
      <q-card>
        <q-card-actions align="right">
          <q-btn flat label="X" color="primary" v-close-popup class="text-h5" />
        </q-card-actions>
        <q-card-section class="q-pa-none text-center">
          <img
            :src="`/img/${state.selectedMenuItem.graphicName}`"
            class="item-image"
            style="height: 150px; width: 100px"
            square
          />
        </q-card-section>
        <q-card-section>
          <div class="text-h6 text-bold text-center text-primary">
            {{ state.selectedMenuItem.productName }} -
            {{ formatCurrency(state.selectedMenuItem.msrp) }}
          </div>
        </q-card-section>
        <q-card-section>
          <div class="text-subtitle2 text-center">
            {{ state.selectedMenuItem.description }}
          </div>
        </q-card-section>

        <q-card-section class="text-center text-positive">
          {{ state.dialogStatus }}
        </q-card-section>
        <q-card-section>
          <div class="row">
            <div class="col-2 q-mr-sm">
              <q-input
                v-model.number="state.qty"
                type="number"
                filled
                style="max-width: 15vw"
                placeholder="qty"
                hint="Qty"
                dense
              />
            </div>
            <div class="col-4 q-mr-sm">
              <q-btn
                color="primary"
                label="Add To Cart"
                :disable="state.qty < 0"
                @click="addToCart()"
              />
            </div>
            <div class="col-3">
              <q-btn color="primary" label="View Cart" @click="viewCart()" />
            </div>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>
<script>
import { formatCurrency } from "../utils/formatutils";
import { reactive, onMounted } from "vue";
import { fetcher } from "../utils/apiutil";
import { useRouter } from "vue-router";
export default {
  setup() {
    const router = useRouter();
    onMounted(() => {
      loadBrands();
    });
    let state = reactive({
      status: "",
      categories: [],
      menuitems: [],
      selectedCategory: {},
      selectedCategoryId: "",
      selectedMenuItem: {},
      dialogStatus: "",
      itemSelected: false,
      qty: 0,
      cart: [],
    });
    const loadBrands = async () => {
      try {
        state.status = "loading brands...";
        const payload = await fetcher(`Brand`);
        if (payload.error === undefined) {
          state.categories = payload;
          state.status = `loaded ${state.categories.length} brands`;
        } else {
          state.status = payload.error;
        }
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };
    const loadMenuitems = async () => {
      try {
        state.selectedCategory = state.categories.find(
          (category) => category.id === state.selectedCategoryId
        );
        state.status = `finding Products for Brand ${state.selectedCategory}...`;
        state.menuitems = await fetcher(`Product/${state.selectedCategory.id}`);
        state.status = `loaded ${state.menuitems.length} products for
${state.selectedCategory.name}`;
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const selectMenuItem = async (menuitemid) => {
      try {
        // q-item click sends us the id, so we need to find the object
        state.selectedMenuItem = state.menuitems.find(
          (item) => item.id === menuitemid
        );
        state.itemSelected = true;
        state.dialogStatus = "";
        if (sessionStorage.getItem("cart")) {
          state.cart = JSON.parse(sessionStorage.getItem("cart"));
        }
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const addToCart = () => {
      let index = -1;
      if (state.cart.length > 0) {
        index = state.cart.findIndex(
          // is item already on the cart
          (item) => item.id === state.selectedMenuItem.id
        );
      }
      if (state.qty > 0) {
        index === -1 // add
          ? state.cart.push({
              id: state.selectedMenuItem.id,
              qty: state.qty,
              item: state.selectedMenuItem,
            })
          : (state.cart[index] = {
              // replace
              id: state.selectedMenuItem.id,
              qty: state.qty,
              item: state.selectedMenuItem,
            });
        state.dialogStatus = `${state.qty} item(s) added`;
      } else {
        index === -1 ? null : state.cart.splice(index, 1); // remove
        state.dialogStatus = `item(s) removed`;
      }
      sessionStorage.setItem("cart", JSON.stringify(state.cart));
      state.qty = 0;
    };

    const viewCart = () => {
      router.push("cart");
    };

    return {
      state,
      loadMenuitems,
      selectMenuItem,
      formatCurrency,
      addToCart,
      viewCart,
    };
  },
};
</script>
