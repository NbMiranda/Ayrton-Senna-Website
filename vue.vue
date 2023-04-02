<template>
  <div v-if="chatContact != null" class="chat__header chat-navbar-component">
    <vs-navbar class="p-4 flex navbar-custom" color="white" type="flat">
      <div class="relative flex mr-4">
        <feather-icon
          icon="MenuIcon"
          class="mr-4 cursor-pointer"
          v-if="isSidebarCollapsed"
          @click.stop="$emit('openContactsSidebar')"
        />
        <vs-avatar
          class="m-0 border-2 border-solid border-white"
          size="40px"
          :src="chatContact.customer.image"
          @click.stop="$emit('showProfileSidebar', chatContact)"
        />
      </div>
      <div
        class="w-full flex"
        :class="{
          'justify-center items-center': chatContact.finished_at === null
        }"
      >
        <div
          :class="{
            'w-6/12': customerLastChat && chatContact.finished_at === null
          }"
        >
          <h6>{{ chatContact.customer.name }}</h6>
          <vs-td>
              <p class="product-phone-number font-medium truncate">{{ maskedPhone }}</p>
          </vs-td>
        </div>
        <div
          class="w-6/12 flex justify-end"
          v-if="customerLastChat && chatContact.finished_at === null && chatContact.is_finishing === false && finishedTickets === false"
        >

          <div class="flex mt-1 mr-4">
            <div v-if="openSearch" class="flex" id="search-input-navbar">
              <vs-input-number
                  min="0"
                  :max="maxValues"
                  v-model="indexSearch"
                  class="w-8 h-8"
                  icon-inc="expand_less"
                  icon-dec="expand_more"
              />

              <vs-input
                  icon-no-border
                  icon="icon-search"
                  icon-pack="feather"
                  v-model="searchQuery"
                  @focusout.prevent="searchMessages"
                  class="input-rounded-full"
                  placeholder="Busque pela mensagem"
              />
            </div>

            <vs-button
                radius
                icon="search"
                type="flat"
                class="ml-3 py-2 px-5"
                @click="openSearch = !openSearch"
            />
          </div>
        
          <div>
            <vs-dropdown v-if="canSee('orders') || canSee('pre-orders') || canSee('products')">
              <vs-button class="btn-drop px-3 mr-4" type="filled" icon="expand_more" icon-after>
                Pedidos
              </vs-button>
  
              <vs-dropdown-menu>

                <vs-dropdown-item v-if="canSee('pre-orders')" @click.prevent="openPreOrdersModal=true">
                  <vs-icon class="mr-2" icon="search" style="-webkit-transform: scaleX(-1); transform: scaleX(-1);">
  
                  </vs-icon>
                  <span>Pré-Pedidos</span>
                </vs-dropdown-item>
                
                <vs-dropdown-item v-if="canSee('products')" @click.prevent="openProductsModal=true">
                  <vs-icon class="mr-2" icon="shopping_basket" style="-webkit-transform: scaleX(-1); transform: scaleX(-1);">
  
                  </vs-icon>
                  <span>Produtos</span>
                </vs-dropdown-item>

                <vs-dropdown-item v-if="canSee('orders')" @click.prevent="openOrdersModal=true">
                  <vs-icon class="mr-2" icon="format_list_bulleted" style="-webkit-transform: scaleX(-1); transform: scaleX(-1);">
  
                  </vs-icon>
                  <span>Pedidos</span>
                </vs-dropdown-item>
                
              </vs-dropdown-menu>
            </vs-dropdown>

            <PreOrdersPopUp 
              :isActive="openPreOrdersModal" 
              @isActiveFalse="openPreOrdersModal=false" 
              @turnIntoOrder="turnIntoOrder"
              @createPreOrder="createPreOrder"/>
            <OrderSummaryPopUp 
              :isActive="openOrderSummaryModal" 
              :dataOfTurnIntoOrder="dataOfTurnIntoOrder" 
              @isActiveFalse="openOrderSummaryModal=false"
              @backToPreOrder="backToPreOrder"
              @editDeliveryOptions="editDeliveryOptions"
              @editPaymentOptions="editPaymentOptions"
              @editProductOptions="editProductOptions"/>
            <ProductsPopUp 
              :isActive="openProductsModal" 
              @isActiveFalse="closeProduct" 
              :backIsVisible="backIsVisible"
              @backToPreOrders="backToPreOrders"
              @addQuantity="addQuantity"
              @backToOrderSummary="backToOrderSummary"
              :dataToUpdate="dataToUpdate"
              :isCreateContext="isCreateContext"/>
            <OrdersPopUp 
              :isActive="openOrdersModal" 
              @isActiveFalse="openOrdersModal=false"/>
            <QuantitypopUp
              :isActive="openQuantityModal" 
              @isActiveFalse="closeQuantity"
              :dataOfProductsToPreOrder="dataOfProductsToPreOrder"
              @backToProducts="backToProducts"
              :quantityList="quantity_list"
              @sendPaymentFlow="sendPaymentFlow"
              :isCreateContext="isCreateContext"
              :dataToUpdate="dataToUpdate"/>
            <PaymentPopUp
              :isActive="openPaymentModal"
              @isActiveFalse="closeModalPayment"
              :dataOfProductsToPreOrder="dataOfProductsToPreOrder"
              @sendDeliveryFlow="sendDeliveryFlow"
              @backToQuantity="backToQuantity"
              :isCreateContext="isCreateContext"
              @backToOrderSummary="backToOrderSummary"
              :dataToUpdate="dataToUpdate"
              @updateOrderPayment="updateOrderPayment"/>
            <DeliveryPopUp
              :isActive="openDeliveryModal"
              @isActiveFalse="closeModalDelivery"
              @backToPayment="backToPayment"
              :dataOfProductsToPreOrder="dataOfProductsToPreOrder"
              @sendToNewPreOrderSummary="sendToNewPreOrderSummary"
              :dataToUpdate="dataToUpdate"
              :isCreateContext="isCreateContext"
              @backToOrderSummary="backToOrderSummary"
              @updateOrderDelivery="updateOrderDelivery"/>
            <PreOrderSummaryPopUp 
              :isActive="openPreOrderSummaryModal"
              @isActiveFalse="openPreOrderSummaryModal=false"
              :dataOfProductsToPreOrder="dataOfProductsToPreOrder"
              :chatContact="chatContact"
              @backToDelivery="backToDelivery"
              :chatTicket="this.chatTicket"/>

          </div>
                    
          <div>
            <vs-dropdown>
              <vs-button class="btn-drop px-3" type="filled" icon="expand_more" icon-after>
                Ações
              </vs-button>

              <vs-dropdown-menu>
                <vs-dropdown-item @click.prevent="openTransferPopup(chatContact)">
                  <vs-icon class="mr-2" icon="reply" style="-webkit-transform: scaleX(-1); transform: scaleX(-1);">

                  </vs-icon>
                  <span>Transferir Atendimento</span>
                </vs-dropdown-item>
                <vs-dropdown-item @click.prevent="openFinalizePopup(chatContact)">
                  <vs-icon class="mr-2" icon="highlight_off"></vs-icon>
                  <span>Encerrar Atendimento</span>
                </vs-dropdown-item>
              </vs-dropdown-menu>
            </vs-dropdown>
          </div>
        </div>
        <div v-else class="w-6/12 flex justify-end"></div>
      </div>
      <vs-spacer></vs-spacer>
    </vs-navbar>
  </div>
</template>

<script>
import { phoneMask } from "@/utils";
import ProductsPopUp from "./products/Products.vue";
import modulePermissions from "@/store/permission/modulePermission";
import moduleRoles from "@/store/role/moduleRole";
import OrdersPopUp from "./orders/Orders.vue";
import PreOrdersPopUp from "./pre-orders/PreOrders.vue";
import OrderSummaryPopUp from "./orders/OrderSummary.vue";
//import NewCustomerPopUp from "./customers/NewCustomer.vue";
import PaymentPopUp from "./payments/Payment.vue";
import DeliveryPopUp from "./delivery/Delivery.vue";
import QuantitypopUp from "./products/Quantity.vue";
import PreOrderSummaryPopUp from "./pre-orders/PreOrderSummary.vue";
import { maskPhoneNumberToBrazilian, hasPermission } from "@/utils";
export default {
  components: {
    ProductsPopUp,
    OrdersPopUp,
    PreOrdersPopUp,
    OrderSummaryPopUp,
    //NewCustomerPopUp,
    PaymentPopUp,
    DeliveryPopUp,
    QuantitypopUp,
    PreOrderSummaryPopUp
  },
  props: {
    chatContact: {
      required: true,
      validator: prop => typeof prop === 'object' || prop === null
    },
    isSidebarCollapsed: {
      type: Boolean,
      required: true
    },
    finishedTickets: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      onlineUsers: [],
      customerServiceClose:false,
      openProductsModal: false,
      canSeeProducts: false,
      openOrdersModal: false,
      openPreOrdersModal: false,
      openOrderSummaryModal: false,
      dataOfTurnIntoOrder: {},
      backIsVisible: false,
      openQuantityModal: false,
      //openNewCustomerModal: false,
      dataOfProductsToPreOrder: {},
      openPaymentModal: false,
      openDeliveryModal: false,
      quantity_list: [],
      openPreOrderSummaryModal: false,
      indexSearch: 0,
      maxValues: 0,
      searchQuery:'',
      openSearch: false,
      transferPopup: false,
      dataToUpdate: {},
      isCreateContext: true
    }
  },
  computed: {
    customerLastChat() {
      return this.chatContact.customerLastChat ? this.chatContact.customerLastChat : null
    },
    chatTicket() {
      return this.chatContact
    },
    instanceId() {
      return this.$store.state.instance.selectedInstance
    },
    maskedPhone() {
      return maskPhoneNumberToBrazilian(this.chatContact.customer.phone_number);
    },
    searchTrim() {
      const text = this.searchQuery.toLowerCase().trim();
      return text.replace(/[^a-zA-Z0-9áéíóúÁÉÍÓÚãõÃÕâêîôûÂÊÎÔÛçÇõñÑ\s]/g, '');
    }
  },
  watch: {
    transferPopup(val) {
      if (val) {
        this.fetchOnlineUsers();
      }
    },
    chatContact(){
      this.cleanSeach();
    },
    searchQuery() {
      this.indexSearch  = 0;
      this.searchMessages();
    },
    openSearch(val) {
      if (!val){
        this.cleanSeach();
      }
    },
    indexSearch() {
      if ( this.searchTrim ){
        this.searchMessages();
      }
    }
  },
  methods: {
    canSee(action) {
      return hasPermission(`${this.$route.name}-${action}`)
    },
    async openFinalizePopup(contact) {
      this.$store.dispatch('chatContact/setChatFinalizePopup', contact);
      await this.sleep(100);
      this.$store.dispatch('chatContact/changeStatusFinalizePopupActive', true);
    },
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    async openTransferPopup(contact) {
      this.$store.dispatch('chatContact/setChatTransferPopup', contact);
      await this.sleep(100);
      this.$store.dispatch('chatContact/changeStatusTransferPopupActive', true);
    },
    turnIntoOrder(value) {
      this.openPreOrdersModal = false;
      this.openOrderSummaryModal = true;
      this.dataOfTurnIntoOrder = value;
    },
    backToPreOrder() {
      this.openOrderSummaryModal = false;
      this.openPreOrdersModal = true;
    },
    createPreOrder(value) {
      this.openPreOrdersModal = false;
      this.openProductsModal = true;
      this.backIsVisible = value.backIsVisible;
    },
    backToPreOrders(value) {
      this.openProductsModal = false;
      this.openPreOrdersModal = true;
      this.backIsVisible = value.backIsVisible;
    },
    closeProduct() {
      this.openProductsModal = false;
      this.backIsVisible = false;
      this.isCreateContext = true;
    },
    addQuantity(value, dataToUpdate) {
      this.openProductsModal = false;
      this.openQuantityModal = true;
      this.dataOfProductsToPreOrder = value;
      this.dataToUpdate = dataToUpdate;
      let lengthOfData = this.dataOfProductsToPreOrder.product === undefined ? this.dataOfProductsToPreOrder.length : this.dataOfProductsToPreOrder.product.length;
      for (let i = 1; i <= lengthOfData; i++) {
        this.quantity_list.push(1);
      }
    },
    backToProducts() {
      this.openQuantityModal = false;
      this.openProductsModal = true;
      this.quantity_list = [];
    },
    sendPaymentFlow(value) {
      this.openQuantityModal = false;
      this.openPaymentModal = true;
      this.dataOfProductsToPreOrder = value;
    },
    backToQuantity() {
      this.openPaymentModal = false;
      this.openQuantityModal = true;
    },
    sendDeliveryFlow(value) {
      this.openPaymentModal = false;
      this.openDeliveryModal = true;
      this.dataOfProductsToPreOrder = value;
    },
    backToPayment() {
      this.openDeliveryModal = false;
      this.openPaymentModal = true;
      this.isCreateContext = true;
    },
    sendToNewPreOrderSummary(value) {
      this.openDeliveryModal = false;
      this.openPreOrderSummaryModal = true;
      this.dataOfProductsToPreOrder = value;
    },
    backToDelivery() {
      this.openPreOrderSummaryModal = false;
      this.openDeliveryModal = true;
    },
    editDeliveryOptions(value) {
      this.openOrderSummaryModal = false;
      this.isCreateContext = false;
      this.dataToUpdate = value;
      this.openDeliveryModal = true;
    },
    closeModalDelivery() {
      this.openDeliveryModal = false;
      this.isCreateContext = true;
    },
    closeModalPayment() {
      this.openPaymentModal = false;
      this.isCreateContext = true;
    },
    backToOrderSummary(value) {
      if (value === 'payment') {
        this.openPaymentModal = false;
      } else if (value === 'product') { 
        this.openProductsModal = false;
        this.backIsVisible = false;
      } else {
        this.openDeliveryModal = false;
      }
      this.isCreateContext = true;
      this.openOrderSummaryModal = true;
    },
    updateOrderDelivery(value) {
      this.dataOfTurnIntoOrder = value;
      this.openDeliveryModal = false;
      this.openOrderSummaryModal = true;
      this.isCreateContext = true
    },
    updateOrderPayment(value) {
      this.dataOfTurnIntoOrder = value;
      this.openPaymentModal = false;
      this.openOrderSummaryModal = true;
      this.isCreateContext = true
    },
    searchMessages() {
      if (!this.searchTrim) { this.cleanSeach(false) }
      this.$emit('searchMessages', this.indexSearch, this.searchTrim);
    },
    editPaymentOptions(value) {
      this.openOrderSummaryModal = false;
      this.dataToUpdate = value;
      this.isCreateContext = false;
      this.openPaymentModal = true;
    },
    editProductOptions(value) {
      this.dataToUpdate = value;
      this.isCreateContext = false;
      this.openOrderSummaryModal = false;
      this.openProductsModal = true;
      this.backIsVisible = true;
    },
    cleanSeach(close = true){
      this.maxValues    = 0;
      this.indexSearch  = 0;
      this.searchQuery  = '';
      if (close) {
        this.openSearch = false;
      }
    },
    closeQuantity() {
      this.isCreateContext = true;
      this.openQuantityModal = false;
      this.backIsVisible = false;
    }
  },
  async mounted() {
    if (!moduleRoles.isRegistered) {
      this.$store.registerModule('role', moduleRoles)
      moduleRoles.isRegistered = true
    }
    if (!modulePermissions.isRegistered) {
      this.$store.registerModule('permission', modulePermissions)
      moduleRoles.isRegistered = true
    }
    await this.$store.dispatch('permission/fetch');
    await this.$store.dispatch(
      'role/infos', 
      this.$store.state.AppActiveUser.role.display_name === 'Atendente' ? 2 : 1
    );
    const listPermissions = await this.$store.state.permission.permissions;
    let idPermission = 0;
    listPermissions.forEach((item) => {
      if (item.display_name === 'Produtos') {
        idPermission = item.id;
      }
    })
    this.canSeeProducts = await this.$store.state.role.roleInfos.has_permissions.includes(idPermission);
  }
}
</script>

<style lang="scss">
.vs-dropdown--item {
  border-radius: 0 !important;
  margin: 0 !important;
  width: 100% !important;
}
.vs-dropdown--item.divider {
  margin-top: 0 !important;
}
.vs-dropdown--item {
    .vs-dropdown--item-link {
      padding-top: 0.75rem !important;
      padding-bottom: 0.75rem !important;
      display: flex !important;
      flex-direction: row !important;
      justify-content: center !important;
      align-items: center !important;
  }
}
.con-vs-dropdown--menu {
  width: 240px !important;
}
.width-popup {
  .vs-popup {
    width: 445px !important;
  }
}
#search-input-navbar {
  .vs-input-number{
    background: none !important;
    flex-direction: column-reverse;
  }
  .vs-input-number-primary button{
    -webkit-transform: none;
    transform: none;
  }
    .vs-input-number--input {
      display: none !important;
    }
}
</style>
