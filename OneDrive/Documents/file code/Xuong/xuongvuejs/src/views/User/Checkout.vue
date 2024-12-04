<script setup>
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

// Khai báo biến
const name = ref("");
const address = ref("");
const phone = ref("");
const email = ref("");
const cartDetail = ref([]);

// Đường dẫn API
const urlCartDetail = "http://localhost/vuejs-server/api.php/cart-detail";
const urlCheckout = "http://localhost/vuejs-server/api.php/check-out";

// Router
const router = useRouter();

// Hàm gọi API giỏ hàng
const callAPI = async () => {
    try {
        let userLogin = localStorage.getItem("userLogin");
        if (!userLogin) {
            alert("Bạn chưa đăng nhập!");
            router.push('/dang-nhap');
            return;
        }

        let user_id = JSON.parse(userLogin).id;
        let response = await axios.get(`${urlCartDetail}?user_id=${user_id}`);
        if (response.status === 200) {
            cartDetail.value = response.data.cart_details;
            console.log(cartDetail.value);
            
        }
    } catch (error) {
        console.error("Lỗi khi gọi API giỏ hàng:", error);
    }
};

// Chuyển đổi giá thành chuỗi định dạng tiền tệ
const convertPrice = (number) => {
    if (!number || isNaN(number)) return "0";
    return number.toLocaleString("vi-VN");
};

// Hàm xử lý thanh toán
const handleSubmit = async () => {
    let user_id = JSON.parse(localStorage.getItem("userLogin")).id;
    console.log(user_id, name.value, address.value, phone.value, email.value);
    
    try {
        let user_id = JSON.parse(localStorage.getItem("userLogin")).id;
        console.log("user_id:", user_id);  // Kiểm tra user_id

        let formData = new FormData();
        formData.append("user_id", user_id);
        formData.append("name", name.value);
        formData.append("address", address.value);
        formData.append("phone", phone.value);
        formData.append("email", email.value);
        let response = await axios.post(urlCheckout, formData);

       // console.log(response);
        
      
        if (response.status === 200) {
            alert("Thanh toán thành công!");
            router.push('/don-hang');
        } else {
            alert("Thanh toán thất bại, vui lòng thử lại!");
        }
    } catch (error) {
        console.error("Lỗi khi xử lý thanh toán:", error);
    }
};


// Gọi API khi component được mount
onMounted(() => {
    callAPI();
});
</script>

<template>
    <div class="row main-website justify-content-center mt-5">
        <div class="col-10">
            <div class="container">
                <div class="row">
                    <div class="col-7">
                        <h4>Thông tin thanh toán</h4>
                        <form @submit.prevent="handleSubmit">
                            <div class="mb-4">
                                <label for="name">Name</label>
                                <input type="text" placeholder="Name" id="name" v-model="name" class="form-control" />
                            </div>
                            <div class="mb-4">
                                <label for="address">Address</label>
                                <input type="text" placeholder="Address" id="address" v-model="address"
                                    class="form-control" />
                            </div>
                            <div class="mb-4">
                                <label for="phone">Phone</label>
                                <input type="text" placeholder="Phone" id="phone" v-model="phone"
                                    class="form-control" />
                            </div>
                            <div class="mb-4">
                                <label for="email">Email</label>
                                <input type="text" placeholder="Email" id="email" v-model="email"
                                    class="form-control" />
                            </div>
                            <button class="btn btn-success">Xác nhận thanh toán</button>
                        </form>
                    </div>
                    <div class="col-5">
                        <h4>Danh sách sản phẩm</h4>
                        <table class="table">
                            <thead>
                                <tr>
                                    <th>STT</th>
                                    <th>Tên sản phẩm</th>
                                    <th>Giá</th>
                                    <th>Số lượng</th>
                                    <th>Thành tiền</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(item, index) in cartDetail" :key="index">
                                    <td>{{ index + 1 }}</td>
                                    <td>{{ item.product_name }}</td>
                                    <td>{{ convertPrice(item.price) }}</td>
                                    <td>{{ item.quantity }}</td>
                                    <td>
                                        {{ convertPrice(Number(item.price) * Number(item.quantity)) }}
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
