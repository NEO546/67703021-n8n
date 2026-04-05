<template>
    <div class="container py-5">
        <div class="row justify-content-center">
            <div class="col-12 col-md-8 col-lg-6">

                <div class="card shadow-lg border-0 rounded-4">

                    <div class="card-header text-center text-white rounded-top-4"
                        style="background: linear-gradient(135deg, #0d6efd, #0dcaf0);">
                        <h4 class="mb-1 fw-bold">📝 ฟอร์มเบิกอุปกรณ์</h4>
                        <small>กรอกข้อมูลเพื่อส่งเข้าสู่ระบบ (n8n Webhook)</small>
                    </div>

                    <div class="card-body p-4">

                        <div v-if="status.message" :class="`alert alert-${status.type} alert-dismissible fade show`">
                            {{ status.message }}
                            <button type="button" class="btn-close" @click="status.message = ''"></button>
                        </div>

                        <form @submit.prevent="submitForm">

                            <div class="mb-3">
                                <label class="form-label">ชื่อ-นามสกุล (ผู้เบิก) *</label>
                                <input type="text" class="form-control" v-model="data.fullname" required placeholder="เช่น ธนิสร แซ่จึง">
                            </div>

                            <div class="mb-3">
                                <label class="form-label">แผนก *</label>
                                <select class="form-select" v-model="data.department" required>
                                    <option value="" disabled>-- เลือกแผนก --</option>
                                    <option value="บุคคล">บุคคล</option>
                                    <option value="บัญชี">บัญชี</option>
                                    <option value="ไอที">ไอที</option>
                                    <option value="การตลาด">การตลาด</option>
                                    <option value="ประชาสัมพันธ์">ประชาสัมพันธ์</option>
                                </select>
                            </div>

                            <div class="mb-3">
                                <label class="form-label">รายการอุปกรณ์ *</label>
                                <select class="form-select" v-model="data.equipment" required>
                                    <option value="" disabled>-- เลือกอุปกรณ์ --</option>
                                    <option value="กรรไกร">กรรไกร</option>
                                    <option value="เทปกาว">เทปกาว</option>
                                    <option value="ปากกา">ปากกา</option>
                                    <option value="แฟ้มเอกสาร">แฟ้มเอกสาร</option>
                                    <option value="คัตเตอร์">คัตเตอร์</option>
                                </select>
                            </div>

                            <div class="mb-3">
                                <label class="form-label">จำนวน *</label>
                                <input type="number" class="form-control" v-model="data.quantity" required min="1" placeholder="ระบุจำนวน">
                            </div>

                            <button class="btn btn-primary w-100 fw-bold mt-2" :disabled="loading">
                                <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
                                {{ loading ? 'กำลังส่งข้อมูล...' : 'บันทึกการเบิกอุปกรณ์' }}
                            </button>

                        </form>

                    </div>

                </div>

            </div>
        </div>
    </div>
</template>

<script setup>
import { reactive, ref } from "vue";

// ตัวแปรเก็บข้อมูลให้ตรงกับฝั่ง n8n ที่รับค่า
const data = reactive({
    fullname: "",
    department: "",
    equipment: "",
    quantity: ""
});

const loading = ref(false);

const status = reactive({
    message: "",
    type: ""
});

const submitForm = async () => {
    loading.value = true;
    status.message = "";

    try {
        // ส่งข้อมูลไปยัง Webhook ของ n8n
        const response = await fetch("http://localhost:5678/webhook/item", {
            method: "POST", // ต้องตั้งค่า n8n โหนด Webhook ให้รับแบบ POST ด้วย
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({
                ...data,
                // แนบวันที่เบิก (เวลาปัจจุบัน) ไปให้ระบบด้วย
                time: new Date().toLocaleString('th-TH') 
            })
        });

        if (!response.ok) {
            throw new Error("Error");
        }

        const result = await response.json();
        console.log(result);

        status.message = "✅ บันทึกข้อมูลการเบิกสำเร็จ!";
        status.type = "success";

        // Reset ฟอร์มหลังส่งสำเร็จ
        data.fullname = "";
        data.department = "";
        data.equipment = "";
        data.quantity = "";

    } catch (error) {
        console.error(error);
        status.message = "❌ เกิดข้อผิดพลาด ไม่สามารถส่งข้อมูลได้";
        status.type = "danger";
    } finally {
        loading.value = false;
    }
};
</script>

<style scoped>
.card {
    transition: 0.3s;
}

.card:hover {
    transform: translateY(-5px);
}

.form-control:focus, .form-select:focus {
    box-shadow: 0 0 0 0.2rem rgba(13, 110, 253, 0.2);
    border-color: #0d6efd;
}
</style>