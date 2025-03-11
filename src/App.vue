<script setup lang="ts">
import { ref, computed } from 'vue'
import { ElMessage } from 'element-plus'

interface LoanRecord {
  downPayment: number
  gjjRate: number
  gjjAmount: number
  businessRate: number
  businessAmount: number
  monthlyPayment: number
  monthlyIncome: number
  monthlyBalance: number
  date: string
  loanYears: number
  totalAmount: number
  totalPayment: number
  totalInterest: number
}

const downPayment = ref(85)
const gjjRate = ref(2.85)
const gjjAmount = ref(60)
const businessRate = ref(3.1)
const businessAmount = ref(50)
const monthlyIncome = ref(7000)
const loanYears = ref(30)
const records = ref<LoanRecord[]>(JSON.parse(localStorage.getItem('loanRecords') || '[]'))

const totalAmount = computed(() => {
  return Math.round((downPayment.value + gjjAmount.value + businessAmount.value) * 10000)
})

const totalPayment = computed(() => {
  return monthlyPayment.value * loanYears.value * 12
})

const totalInterest = computed(() => {
  return totalPayment.value - (gjjAmount.value + businessAmount.value) * 10000
})

const monthlyPayment = computed(() => {
  if (gjjAmount.value <= 0 && businessAmount.value <= 0) return 0

  const gjjMonthlyRate = gjjRate.value / 100 / 12
  const businessMonthlyRate = businessRate.value / 100 / 12
  const months = loanYears.value * 12

  // 将万转换为元进行计算
  const gjjMonthly = gjjAmount.value > 0
    ? (gjjAmount.value * 10000 * gjjMonthlyRate * Math.pow(1 + gjjMonthlyRate, months)) / (Math.pow(1 + gjjMonthlyRate, months) - 1)
    : 0

  const businessMonthly = businessAmount.value > 0
    ? (businessAmount.value * 10000 * businessMonthlyRate * Math.pow(1 + businessMonthlyRate, months)) / (Math.pow(1 + businessMonthlyRate, months) - 1)
    : 0

  return Math.round(gjjMonthly + businessMonthly)
})

const monthlyBalance = computed(() => {
  return Math.round(monthlyIncome.value - monthlyPayment.value)
})

const saveRecord = () => {
  if (monthlyPayment.value <= 0) {
    ElMessage.warning('请输入有效的贷款金额')
    return
  }

  const record: LoanRecord = {
    downPayment: downPayment.value,
    gjjRate: gjjRate.value,
    gjjAmount: gjjAmount.value,
    businessRate: businessRate.value,
    businessAmount: businessAmount.value,
    monthlyPayment: monthlyPayment.value,
    monthlyIncome: monthlyIncome.value,
    monthlyBalance: monthlyBalance.value,
    date: new Date().toLocaleString(),
    loanYears: loanYears.value,
    totalAmount: totalAmount.value,
    totalPayment: totalPayment.value,
    totalInterest: totalInterest.value
  }

  records.value.unshift(record)
  localStorage.setItem('loanRecords', JSON.stringify(records.value))
  ElMessage.success('记录已保存')
}

const clearRecords = () => {
  records.value = []
  localStorage.removeItem('loanRecords')
  ElMessage.success('记录已清空')
}

const deleteRecord = (index: number) => {
  records.value.splice(index, 1)
  localStorage.setItem('loanRecords', JSON.stringify(records.value))
  ElMessage.success('记录已删除')
}

const handleDragEnd = () => {
  localStorage.setItem('loanRecords', JSON.stringify(records.value))
  ElMessage.success('排序已保存')
}
</script>

<template>
  <div class="container">
    <h1>房贷计算器</h1>

    <el-card class="calculator">
      <el-form label-width="120px">
        <el-form-item label="首付金额">
          <el-input-number v-model="downPayment" :min="0" :step="1" :precision="0" />
          <span class="unit">万</span>
        </el-form-item>

        <el-form-item label="贷款年限">
          <el-input-number v-model="loanYears" :min="1" :max="30" :step="1" :precision="0" />
          <span class="unit">年</span>
        </el-form-item>

        <el-form-item label="公积金贷款">
          <el-input-number v-model="gjjAmount" :min="0" :step="1" :precision="0" />
          <span class="unit">万</span>
          <span class="label">利率：</span>
          <el-input-number v-model="gjjRate" :min="0" :max="100" :step="0.1" :precision="2" class="rate-input" />
          <span class="unit">%</span>
        </el-form-item>

        <el-form-item label="商业贷款">
          <el-input-number v-model="businessAmount" :min="0" :step="1" :precision="0" />
          <span class="unit">万</span>
          <span class="label">利率：</span>
          <el-input-number v-model="businessRate" :min="0" :max="100" :step="0.1" :precision="2" class="rate-input" />
          <span class="unit">%</span>
        </el-form-item>

        <el-form-item label="月收入">
          <el-input-number v-model="monthlyIncome" :min="0" :step="100" :precision="0" />
          <span class="unit">元</span>
        </el-form-item>

        <el-form-item>
          <div class="result">
            <span class="label">房款总额：</span>
            <span style="color:red" class="amount">{{ Math.round(totalAmount / 10000) }}</span>
            <span class="unit">万</span>
          </div>
        </el-form-item>

        <el-form-item>
          <div class="result">
            <span class="label">月供：</span>
            <span class="amount">{{ monthlyPayment }}</span>
            <span class="unit">元</span>
            <span class="label" style="margin-left: 20px;">每月剩余：</span>
            <span class="amount" :style="{ color: monthlyBalance < 0 ? '#F56C6C' : '#67C23A' }">{{ monthlyBalance}}</span>
            <span class="unit">元</span>
          </div>
        </el-form-item>

        <el-form-item>
          <div class="result">
            <span class="label">还款总额：</span>
            <span class="amount">{{ totalPayment }}</span>
            <span class="unit">元</span>
            <span class="label" style="margin-left: 20px;">总利息：</span>
            <span class="amount">{{ totalInterest }}</span>
            <span class="unit">元</span>
          </div>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="saveRecord">保存记录</el-button>
          <el-button @click="clearRecords">清空记录</el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <el-card class="history">
      <template #header>
        <div class="card-header">
          <span>历史记录</span>
        </div>
      </template>
      <el-table :data="records" row-key="date" @row-end="handleDragEnd" row-draggable>
        <el-table-column prop="loanYears" label="贷款年限" width="auto">
          <template #default="scope">
            {{ scope.row.loanYears }} 年
          </template>
        </el-table-column>
        <el-table-column prop="totalAmount" label="总房款" width="100">
          <template #default="scope">
            {{ Math.round(scope.row.totalAmount / 10000) }} 万
          </template>
        </el-table-column>
        <el-table-column prop="downPayment" label="首付" width="auto">
          <template #default="scope">
            {{ scope.row.downPayment }} 万
          </template>
        </el-table-column>
        <el-table-column prop="gjjAmount" label="公积金贷款" width="100">
          <template #default="scope">
            {{ scope.row.gjjAmount }} 万
          </template>
        </el-table-column>
        <el-table-column prop="businessAmount" label="商业贷款" width="auto">
          <template #default="scope">
            {{ scope.row.businessAmount }} 万
          </template>
        </el-table-column>
        <el-table-column prop="monthlyPayment" label="月供" width="auto">
          <template #default="scope">
            <span :style="{ color: '#409EFF', fontWeight: 'bold' }">
              {{ scope.row.monthlyPayment }} 元
            </span>
          </template>
        </el-table-column>
        <el-table-column prop="totalInterest" label="总利息" width="100">
          <template #default="scope">
            {{ scope.row.totalInterest }} 元
          </template>
        </el-table-column>
        <el-table-column prop="monthlyIncome" label="月收入" width="auto">
          <template #default="scope">
            {{ scope.row.monthlyIncome }} 元
          </template>
        </el-table-column>
        <el-table-column prop="monthlyBalance" label="每月剩余" width="110">
          <template #default="scope">
            <span :style="{ color: scope.row.monthlyBalance < 0 ? '#F56C6C' : '#67C23A' }">
              {{ scope.row.monthlyBalance }} 元
            </span>
          </template>
        </el-table-column>
        
        <el-table-column label="操作" width="auto">
          <template #default="scope">
            <el-button type="danger" size="small" @click="deleteRecord(scope.$index)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<style scoped>
.container {
  max-width: 100%;
  margin: 0 auto;
  padding: 0px;
}

@media screen and (max-width: 768px) {
  .container {
    padding: 10px 5px;
  }

  .el-form-item {
    margin-bottom: 15px;
  }

  .el-form-item__label {
    padding: 0 5px;
    line-height: 30px;
  }

  .el-input-number {
    width: 120px;
  }

  .rate-input {
    width: 80px;
    margin-left: 10px;
  }

  .unit {
    margin: 0 5px;
    font-size: 14px;
  }

  .result {
    font-size: 16px;
  }

  .result .amount {
    font-size: 20px;
  }

  .el-table {
    width: 100%;
    overflow-x: auto;
    white-space: nowrap;
  }
}

h1 {
  text-align: center;
  color: #409EFF;
  margin-bottom: 30px;
}

.calculator {
  margin-bottom: 30px;
}

.unit {
  margin: 0 10px;
}

.rate-input {
  width: 120px;
  margin-left: 20px;
}

.result {
  font-size: 18px;
  color: #409EFF;
}

.result .amount {
  font-size: 24px;
  font-weight: bold;
  margin: 0 10px;
}

.history {
  margin-top: 20px;
}

.card-header {
  font-weight: bold;
}
</style>
