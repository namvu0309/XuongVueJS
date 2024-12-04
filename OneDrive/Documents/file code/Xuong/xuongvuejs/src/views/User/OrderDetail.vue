<script setup>
import { useRoute } from "vue-router";
import axios from "axios";
import { ref, onMounted } from "vue";

let urlOrderDetail = "http://localhost/vuejs-server/api.php/order_detail";
const route = useRoute();

const convertPrice = (number) => {
    return number.toLocaleString("vi-VN");
};

const orderDetail = ref([]);

const getOrderDetail = async () => {
    try {
        let res = await axios.get(
            urlOrderDetail + "/?order_id=" + route.params.order_id
        );
        console.log(res.data);
        if (res.status == 200) {
            orderDetail.value = res.data;
        }
    } catch (error) {
        console.log("Lỗi call api", error);
    }
};

onMounted(() => {
    getOrderDetail();
});
</script>
<template>
    <main class="container my-5">
        <!-- Tiêu đề -->
        <div class="text-center mb-5">
            <h1 class="fw-bold">Chi Tiết Đơn Hàng</h1>
        </div>

        <!-- Danh sách sản phẩm -->
        <div class="card shadow">
            <div class="card-header bg-success text-white">
                <h5 class="mb-0">Danh Sách Sản Phẩm</h5>
            </div>
            <div class="card-body">
                <div class="table-responsive">
                    <table class="table table-bordered align-middle">
                        <thead class="table-light">
                            <tr>
                                <th>#</th>
                                <th>Hình Ảnh</th>
                                <th>Tên Sản Phẩm</th>
                                <th>Số Lượng</th>
                                <th>Giá</th>
                                <th>Thành Tiền</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(item, index) in orderDetail" :key="index">
                                <td>{{ index + 1 }}</td>
                                <td>
                                    <img :src="'http://localhost/vuejs-server/uploads/product/' +
                                        item.product_image
                                        " alt="Sản phẩm" class="img-fluid rounded" style="width: 80px; height: 80px; object-fit: cover" />
                                </td>
                                <td>{{ item.product_name }}</td>
                                <td>{{ item.quantity }}</td>
                                <td>{{ convertPrice(Number(item.price)) }} VNĐ</td>
                                <td>
                                    {{ convertPrice(Number(item.price * item.quantity)) }} VNĐ
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </main>
</template>

<style scoped>
table th,
table td {
    text-align: center;
    vertical-align: middle;
}
</style>