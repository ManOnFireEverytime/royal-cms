<template>
  <div class="orders">
    <div class="header d-flex justify-content-between align-items-center">
      <h5>Orders</h5>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="text-center py-4">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="alert alert-danger" role="alert">
      {{ error }}
      <button class="btn btn-outline-danger btn-sm ms-2" @click="fetchOrders">
        Retry
      </button>
    </div>

    <!-- Orders List -->
    <div v-else class="orders-container">
      <!-- Orders Table - Desktop -->
      <div class="table-responsive d-none d-md-block">
        <table class="table orders-table">
          <thead>
            <tr>
              <th>Order ID</th>
              <th>Reference</th>
              <th>Customer</th>
              <th>Email</th>
              <th>Items</th>
              <th>Total</th>
              <th>Payment Status</th>
              <th>Date</th>
              <th class="text-end">Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="order in orders" :key="order.id">
              <td>#{{ order.id }}</td>
              <td>{{ order.reference }}</td>
              <td>
                {{ order.customer_first_name }} {{ order.customer_last_name }}
              </td>
              <td>{{ order.customer_email }}</td>
              <td>{{ order.item_count }}</td>
              <td>₦{{ formatNumber(order.total) }}</td>
              <td>
                <span
                  :class="getPaymentStatusClass(order.payment_status)"
                  class="badge"
                >
                  {{ order.payment_status }}
                </span>
              </td>
              <td>{{ formatDate(order.created_at) }}</td>
              <td class="text-end">
                <button
                  class="btn btn-sm btn-outline-primary"
                  @click="viewOrder(order)"
                >
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    width="16"
                    height="16"
                    fill="currentColor"
                    class="bi bi-eye"
                    viewBox="0 0 16 16"
                  >
                    <path
                      d="M16 8s-3-5.5-8-5.5S0 8 0 8s3 5.5 8 5.5S16 8 16 8M1.173 8a13 13 0 0 1 1.66-2.043C4.12 4.668 5.88 3.5 8 3.5s3.879 1.168 5.168 2.457A13 13 0 0 1 14.828 8q-.086.13-.195.288c-.335.48-.83 1.12-1.465 1.755C11.879 11.332 10.119 12.5 8 12.5s-3.879-1.168-5.168-2.457A13 13 0 0 1 1.172 8z"
                    />
                    <path
                      d="M8 5.5a2.5 2.5 0 1 0 0 5 2.5 2.5 0 0 0 0-5M4.5 8a3.5 3.5 0 1 1 7 0 3.5 3.5 0 0 1-7 0"
                    />
                  </svg>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Orders Cards - Mobile -->
      <div class="d-md-none">
        <div
          v-for="order in orders"
          :key="order.id"
          class="order-card"
          @click="viewOrder(order)"
        >
          <div class="order-card-header">
            <div>
              <strong>#{{ order.id }}</strong>
              <div class="text-muted small">{{ order.reference }}</div>
            </div>
            <span
              :class="getPaymentStatusClass(order.payment_status)"
              class="badge"
            >
              {{ order.payment_status }}
            </span>
          </div>
          <div class="order-card-body">
            <div class="order-card-row">
              <span class="label">Customer:</span>
              <span class="value"
                >{{ order.customer_first_name }}
                {{ order.customer_last_name }}</span
              >
            </div>
            <div class="order-card-row">
              <span class="label">Email:</span>
              <span class="value">{{ order.customer_email }}</span>
            </div>
            <div class="order-card-row">
              <span class="label">Items:</span>
              <span class="value">{{ order.item_count }}</span>
            </div>
            <div class="order-card-row">
              <span class="label">Total:</span>
              <span class="value fw-bold"
                >₦{{ formatNumber(order.total) }}</span
              >
            </div>
            <div class="order-card-row">
              <span class="label">Date:</span>
              <span class="value small">{{
                formatDate(order.created_at)
              }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- No Orders State -->
      <div v-if="orders.length === 0" class="text-center py-4">
        <p>No orders found.</p>
      </div>
    </div>

    <!-- Order Details Modal -->
    <div
      v-if="showOrderModal"
      class="modal-backdrop"
      @click="showOrderModal = false"
    >
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h4>Order Details - #{{ selectedOrder?.id }}</h4>
          <button class="btn-close" @click="showOrderModal = false">
            &times;
          </button>
        </div>
        <div class="modal-body" v-if="selectedOrder">
          <div class="info-section">
            <div class="info-block">
              <h6>Customer Information</h6>
              <p>
                <strong>Name:</strong> {{ selectedOrder.customer_first_name }}
                {{ selectedOrder.customer_last_name }}
              </p>
              <p><strong>Email:</strong> {{ selectedOrder.customer_email }}</p>
              <p><strong>Phone:</strong> {{ selectedOrder.customer_phone }}</p>
            </div>
            <div class="info-block">
              <h6>Shipping Address</h6>
              <p>{{ selectedOrder.shipping_address }}</p>
              <p>
                {{ selectedOrder.shipping_city }},
                {{ selectedOrder.shipping_state }}
              </p>
              <p>{{ selectedOrder.shipping_country }}</p>
            </div>
          </div>

          <hr />

          <div class="info-section single-column">
            <div class="info-block">
              <h6>Order Summary</h6>
              <p><strong>Reference:</strong> {{ selectedOrder.reference }}</p>
              <p>
                <strong>Payment Reference:</strong>
                {{ selectedOrder.payment_reference }}
              </p>
              <p>
                <strong>Payment Status:</strong>
                <span
                  :class="getPaymentStatusClass(selectedOrder.payment_status)"
                  class="badge ms-2"
                >
                  {{ selectedOrder.payment_status }}
                </span>
              </p>
              <p>
                <strong>Order Date:</strong>
                {{ formatDate(selectedOrder.created_at) }}
              </p>
            </div>
          </div>

          <hr />

          <div class="items-section">
            <h6>Order Items</h6>

            <!-- Desktop Table View -->
            <div class="table-responsive d-none d-md-block">
              <table class="table">
                <thead>
                  <tr>
                    <th>Product</th>
                    <th>Color</th>
                    <th>Size</th>
                    <th>Quantity</th>
                    <th>Price</th>
                    <th>Subtotal</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in orderItems" :key="item.id">
                    <td>{{ item.product_name }}</td>
                    <td>{{ item.color || "N/A" }}</td>
                    <td>{{ item.size || "N/A" }}</td>
                    <td>{{ item.quantity }}</td>
                    <td>₦{{ formatNumber(item.price) }}</td>
                    <td>₦{{ formatNumber(item.price * item.quantity) }}</td>
                  </tr>
                </tbody>
              </table>
            </div>

            <!-- Mobile Card View -->
            <div class="d-md-none">
              <div v-for="item in orderItems" :key="item.id" class="item-card">
                <div class="item-card-header">
                  <strong>{{ item.product_name }}</strong>
                </div>
                <div class="item-card-body">
                  <div class="item-detail">
                    <span class="label">Color:</span>
                    <span>{{ item.color || "N/A" }}</span>
                  </div>
                  <div class="item-detail">
                    <span class="label">Size:</span>
                    <span>{{ item.size || "N/A" }}</span>
                  </div>
                  <div class="item-detail">
                    <span class="label">Quantity:</span>
                    <span>{{ item.quantity }}</span>
                  </div>
                  <div class="item-detail">
                    <span class="label">Price:</span>
                    <span>₦{{ formatNumber(item.price) }}</span>
                  </div>
                  <div class="item-detail fw-bold">
                    <span class="label">Subtotal:</span>
                    <span>₦{{ formatNumber(item.price * item.quantity) }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <hr />

          <div class="totals-section">
            <div class="totals-row">
              <span>Subtotal:</span>
              <span>₦{{ formatNumber(selectedOrder.subtotal) }}</span>
            </div>
            <div class="totals-row">
              <span>Delivery Fee:</span>
              <span>₦{{ formatNumber(selectedOrder.delivery_fee) }}</span>
            </div>
            <div class="totals-row total">
              <span>Total:</span>
              <span>₦{{ formatNumber(selectedOrder.total) }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const orders = ref([]);
const loading = ref(true);
const error = ref(null);
const showOrderModal = ref(false);
const selectedOrder = ref(null);
const orderItems = ref([]);

const fetchOrders = async () => {
  try {
    loading.value = true;
    error.value = null;

    const response = await fetch(
      "https://backend.royalgangchamber.com/getAllOrders.php"
    );
    const result = await response.json();

    if (result.status === "success") {
      orders.value = result.data;
    } else {
      throw new Error(result.message || "Failed to fetch orders");
    }
  } catch (err) {
    error.value = err.message;
    console.error("Error fetching orders:", err);
  } finally {
    loading.value = false;
  }
};

const viewOrder = async (order) => {
  try {
    const response = await fetch(
      `https://backend.royalgangchamber.com/getOrder.php?id=${order.id}`
    );
    const result = await response.json();

    if (result.status === "success") {
      selectedOrder.value = result.data;
      orderItems.value = result.data.items || [];
      showOrderModal.value = true;
      document.body.style.overflow = "hidden";
    } else {
      throw new Error(result.message || "Failed to fetch order details");
    }
  } catch (err) {
    console.error("Error fetching order details:", err);
    alert("Failed to load order details");
  }
};

const closeModal = () => {
  showOrderModal.value = false;
  document.body.style.overflow = "auto";
};

const formatNumber = (number) => {
  return new Intl.NumberFormat().format(number);
};

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString("en-US", {
    year: "numeric",
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
};

const getPaymentStatusClass = (status) => {
  const statusLower = status?.toLowerCase() || "";
  if (statusLower === "success" || statusLower === "paid") {
    return "bg-success";
  } else if (statusLower === "pending") {
    return "bg-warning";
  } else if (statusLower === "failed") {
    return "bg-danger";
  }
  return "bg-secondary";
};

onMounted(() => {
  fetchOrders();
});
</script>

<style scoped>
/* Orders Table - Desktop */
.orders-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0 8px;
}

.orders-table th,
.orders-table td {
  vertical-align: middle;
  padding: 10px 15px;
}

.orders-table th {
  border-bottom: 2px solid #dee2e6;
}

.orders-table td {
  background-color: #f8f9fa;
}

.text-end {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 10px;
}

/* Mobile Order Cards */
.order-card {
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 15px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.order-card:hover {
  background-color: #e9ecef;
}

.order-card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 12px;
  padding-bottom: 12px;
  border-bottom: 1px solid #dee2e6;
}

.order-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.order-card-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
}

.order-card-row .label {
  color: #6c757d;
  font-weight: 500;
}

.order-card-row .value {
  text-align: right;
  max-width: 60%;
  word-break: break-word;
}

/* Modal Backdrop */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
  padding: 20px;
  overflow-y: auto;
}

/* Modal Content */
.modal-content {
  background-color: white;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  width: 100%;
  max-width: 950px;
  max-height: 90vh;
  overflow-y: auto;
  margin: auto;
  position: relative;
}

/* Modal Header */
.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 25px;
  border-bottom: 1px solid #e0e0e0;
  background-color: #f8f9fa;
  border-radius: 12px 12px 0 0;
  position: sticky;
  top: 0;
  z-index: 1;
}

.modal-header h4 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
}

.btn-close {
  background: none;
  border: none;
  font-size: 28px;
  cursor: pointer;
  padding: 0;
  width: 35px;
  height: 35px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #666;
  transition: color 0.2s;
  flex-shrink: 0;
}

.btn-close:hover {
  color: #000;
}

/* Modal Body */
.modal-body {
  padding: 25px;
}

.modal-body hr {
  margin: 25px 0;
  border-top: 1px solid #e0e0e0;
}

.modal-body h6 {
  font-weight: 600;
  margin-bottom: 15px;
  font-size: 16px;
  color: #333;
}

.modal-body p {
  margin-bottom: 10px;
  line-height: 1.6;
  font-size: 14px;
}

/* Info Sections */
.info-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 25px;
  margin-bottom: 20px;
}

.info-section.single-column {
  grid-template-columns: 1fr;
}

.info-block {
  min-width: 0;
}

/* Items Section */
.items-section {
  margin-bottom: 20px;
}

.items-section .table {
  margin-top: 15px;
  font-size: 14px;
}

.items-section .table th {
  background-color: #f8f9fa;
  font-weight: 600;
  padding: 12px;
}

.items-section .table td {
  padding: 12px;
}

/* Mobile Item Cards */
.item-card {
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 12px;
}

.item-card-header {
  margin-bottom: 12px;
  padding-bottom: 10px;
  border-bottom: 1px solid #dee2e6;
  font-size: 15px;
}

.item-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.item-detail {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
}

.item-detail .label {
  color: #6c757d;
  font-weight: 500;
}

/* Totals Section */
.totals-section {
  display: flex;
  flex-direction: column;
  gap: 10px;
  align-items: flex-end;
}

.totals-row {
  display: flex;
  gap: 30px;
  justify-content: space-between;
  min-width: 250px;
  font-size: 14px;
}

.totals-row.total {
  font-size: 18px;
  font-weight: 600;
  padding-top: 10px;
  border-top: 2px solid #dee2e6;
}

/* Badge */
.badge {
  padding: 6px 12px;
  border-radius: 4px;
  color: white;
  font-size: 12px;
  font-weight: 500;
}

/* Responsive Styles */
@media screen and (max-width: 768px) {
  .modal-backdrop {
    padding: 10px;
    align-items: flex-start;
  }

  .modal-content {
    max-height: calc(100vh - 20px);
    border-radius: 8px;
    margin-top: 10px;
    margin-bottom: 10px;
  }

  .modal-header {
    padding: 15px;
    border-radius: 8px 8px 0 0;
  }

  .modal-header h4 {
    font-size: 16px;
  }

  .modal-body {
    padding: 15px;
  }

  .info-section {
    grid-template-columns: 1fr;
    gap: 20px;
  }

  .modal-body h6 {
    font-size: 15px;
    margin-bottom: 12px;
  }

  .modal-body p {
    font-size: 13px;
  }

  .totals-row {
    min-width: 100%;
    font-size: 13px;
    gap: 15px;
  }

  .totals-row.total {
    font-size: 16px;
  }
}

@media screen and (max-width: 500px) {
  .modal-content {
    width: calc(100vw - 20px);
  }

  .modal-header {
    padding: 12px 15px;
  }

  .modal-header h4 {
    font-size: 15px;
  }

  .btn-close {
    width: 30px;
    height: 30px;
    font-size: 24px;
  }

  .modal-body {
    padding: 12px;
  }

  .modal-body hr {
    margin: 20px 0;
  }
}
</style>
