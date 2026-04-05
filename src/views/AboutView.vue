<template>
  <div class="container py-5">

    <div class="text-center mb-4">
      <h2 class="fw-bold text-primary">📋 ระบบแสดงข้อมูลการเบิกอุปกรณ์</h2>
      <p class="text-muted">ข้อมูลจาก n8n Webhook API</p>
    </div>

    <div class="card shadow-lg border-0 rounded-4">
      <div class="card-body">

        <div class="d-flex justify-content-between align-items-center mb-3">
          <h5 class="mb-0">รายการการเบิกทั้งหมด</h5>
          <button class="btn btn-primary" @click="fetchData">
            🔄 โหลดข้อมูล
          </button>
        </div>

        <div v-if="loading" class="text-center my-4">
          <div class="spinner-border text-primary"></div>
          <p class="mt-2">กำลังโหลดข้อมูล...</p>
        </div>

        <div class="table-responsive" v-if="users.length">
          <table class="table table-hover align-middle text-center">
            <thead class="table-primary">
              <tr>
                <th>#</th>
                <th>ชื่อ-นามสกุล (ผู้เบิก)</th>
                <th>แผนก</th>
                <th>รายการอุปกรณ์</th>
                <th>จำนวน</th>
                <th>วันที่เบิก</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in users" :key="index">
                <td>{{ index + 1 }}</td>
                <td>{{ item.fullname }}</td>
                <td>{{ item.department }}</td>
                <td>{{ item.equipment }}</td>
                <td>{{ item.quantity }}</td>
                <td>{{ item.time }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-else-if="!loading" class="text-center text-muted py-4">
          ❌ ไม่มีข้อมูล
        </div>

      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const users = ref([])
const loading = ref(false)

const fetchData = async () => {
  loading.value = true
  try {
    // ดึงข้อมูลจาก Webhook URL ของ n8n (ตรวจสอบว่า n8n รันอยู่และเปิด Active)
    const response = await fetch('http://localhost:5678/webhook/item')
    const data = await response.json()
    users.value = data
  } catch (error) {
    console.error('Error fetching data:', error)
  }
  loading.value = false
}

// โหลดข้อมูลทันทีเมื่อเปิดหน้าเว็บ
onMounted(() => {
  fetchData()
})
</script>

<style>
body {
  background-color: #f8f9fa;
}
</style>