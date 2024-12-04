<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";

// Biến lưu thông tin người dùng
const name = ref("");
const address = ref("");
const phone = ref("");
const email = ref("");

// Vue Router
const router = useRouter();

// Chi tiết giỏ hàng
const order = ref([]);

// API URLs
const urlCartDetail = "http://localhost/vuejs-server/api.php/show_order";
const urlCheckout = "http://localhost/vuejs-server/api.php/check-out";

// Gọi API để lấy thông tin giỏ hàng
const callAPI = async () => {
    try {
        const userLogin = localStorage.getItem("userLogin");
        if (!userLogin) {
            console.error("Người dùng chưa đăng nhập.");
            return;
        }

        const user_id = JSON.parse(userLogin).id;
        const response = await axios.get(`${urlCartDetail}?user_id=${user_id}`);
        if (response.status === 200) {
            order.value = response.data;
            console.log(order.value);
        }
    } catch (error) {
        console.error("Lỗi khi lấy dữ liệu đơn hàng:", error);
    }
};

// Gọi API khi component được mount
onMounted(() => {
    callAPI();
});

// Chuyển đổi định dạng ngày sang dd/mm/yyyy
function convertToDateFormat(input) {
    if (!input) {
        console.warn("Ngày đặt hàng không hợp lệ:", input);
        return "Ngày không hợp lệ";
    }
    const dateParts = input.split("-");
    if (dateParts.length !== 3) {
        console.warn("Định dạng ngày sai:", input);
        return "Ngày không hợp lệ";
    }
    const year = dateParts[0];
    const month = dateParts[1];
    const day = dateParts[2];
    return `${day}/${month}/${year}`;
}

// Chuyển đổi giá sang định dạng tiền tệ
function convertPrice(amount) {
    return new Intl.NumberFormat('vi-VN', { style: 'currency', currency: 'VND' }).format(amount);
}

// Xem chi tiết đơn hàng
function viewOrder(item) {
    router.push({ name: 'order-details', params: { orderId: item.id } });
}

// Hủy đơn hàng
function cancelOrder(item) {
    axios.post('http://localhost/vuejs-server/api.php/cancel-order', { orderId: item.id })
        .then(response => {
            if (response.status === 200) {
                console.log("Hủy đơn hàng thành công.");
                callAPI(); // Cập nhật lại danh sách đơn hàng
            }
        })
        .catch(error => {
            console.error("Lỗi khi hủy đơn hàng:", error);
        });
}
</script>

<template>
    <div class="row main-website justify-content-center mt-5">
        <div class="col-10">
            <table class="table table-bordered">
                <thead>
                    <tr>
                        <th>STT</th>
                        <th>Ngày đặt hàng</th>
                        <th>Trạng thái</th>
                        <th>Tổng tiền</th>
                        <th>Thao tác</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in order" :key="index">
                        <td>{{ index + 1 }}</td>
                        <td>{{ convertToDateFormat(item.order_date) }}</td>
                        <td>
                            <span v-if="item.status === 1" class="badge text-bg-warning">
                                Chưa xác nhận
                            </span>
                            <span v-else-if="item.status === 2" class="badge text-bg-info">
                                Đã xác nhận
                            </span>
                            <span v-else-if="item.status === 3" class="badge text-bg-danger">
                                Đã hủy
                            </span>
                            <span v-else-if="item.status === 4" class="badge text-bg-primary">
                                Đang giao hàng
                            </span>
                            <span v-else class="badge text-bg-success">Đã giao hàng</span>
                        </td>
                        <td>{{ convertPrice(item.total_price) }}</td>
                        <td>
                            <button @click="viewOrder(item)" class="btn btn-info">
                                Xem chi tiết
                            </button>
                            <button @click="cancelOrder(item)" class="btn btn-danger">
                                Hủy đơn
                            </button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>
