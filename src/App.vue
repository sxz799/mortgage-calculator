<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
// @ts-ignore
import Sortable from 'sortablejs'

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

const downPayment = ref(80)
const gjjRate = ref(2.6)
const gjjAmount = ref(60)
const businessRate = ref(3.05)
const businessAmount = ref(60)
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

onMounted(() => {
  const el = document.querySelector('.el-table__body-wrapper tbody')
  if (el) {
    new Sortable(el, {
      animation: 150,
      onEnd({ newIndex, oldIndex }: { newIndex: number; oldIndex: number }) {
        if (newIndex === undefined || oldIndex === undefined) return
        const recordsCopy = [...records.value]
        const [moved] = recordsCopy.splice(oldIndex, 1)
        recordsCopy.splice(newIndex, 0, moved)
        records.value = recordsCopy
        localStorage.setItem('loanRecords', JSON.stringify(records.value))
      }
    })
  }
})

</script>

<template>
  <div class="container">
    <h1>房贷计算器</h1>


      <el-form label-width="auto">
        <el-form-item label="首付金额">
          <el-input-number v-model="downPayment" :min="0" :step="5" :precision="0" />
          <span class="unit">万</span>
        </el-form-item>

        <el-form-item label="贷款年限">
          <el-input-number v-model="loanYears" :min="1" :max="30" :step="1" :precision="0" />
          <span class="unit">年</span>
        </el-form-item>

        <el-form-item label="公积金贷款">
          <el-input-number v-model="gjjAmount" :min="0" :step="1" :precision="0" />
          <span class="unit">万</span>

        </el-form-item>

        <el-form-item label="利率：">
          <el-input-number v-model="gjjRate" :min="0" :max="100" :step="0.05" :precision="2" />
          <span class="unit">%</span>
        </el-form-item>

        <el-form-item label="商业贷款">
          <el-input-number v-model="businessAmount" :min="0" :step="1" :precision="0" />
          <span class="unit">万</span>

        </el-form-item>
        <el-form-item label="利率：">
          <el-input-number v-model="businessRate" :min="0" :max="100" :step="0.05" :precision="2" />
          <span class="unit">%</span>
        </el-form-item>

        <el-form-item label="月收入">
          <el-input-number v-model="monthlyIncome" :min="0" :step="100" :precision="0" />
          <span class="unit">元</span>
        </el-form-item>
        <el-button type="primary" @click="saveRecord">保存记录</el-button>
        <el-button @click="clearRecords">清空记录</el-button>
      </el-form>

      <div class="result">
        <span class="label">房款总额：</span>
        <span style="color:red" class="amount">{{ Math.round(totalAmount / 10000) }}</span>
        <span class="unit">万</span>
      </div>



      <div class="result">
        <span class="label">月供：</span>
        <span class="amount">{{ monthlyPayment }}</span>
        <span class="unit">元</span>

      </div>
      <div class="result">
        <span class="label" style="margin-left: 20px;">每月剩余：</span>
        <span class="amount" :style="{ color: monthlyBalance < 0 ? '#F56C6C' : '#67C23A' }">{{ monthlyBalance }}</span>
        <span class="unit">元</span>
      </div>


      <div class="result">
        <span class="label">还款总额：</span>
        <span class="amount">{{ totalPayment }}</span>
        <span class="unit">元</span>

      </div>

      <div class="result">
        <span class="label" style="margin-left: 20px;">总利息：</span>
        <span class="amount">{{ totalInterest }}</span>
        <span class="unit">元</span>
      </div>








    <el-table :data="records" style="width: 100%">
      <el-table-column type="index" label="序号" min-width="10" />
      <el-table-column type="expand" label="详情" :default-expand-all="true" min-width="10">
        <template #default="scope">
          <div style="width: 100%;">
            <p>贷款年限: {{ scope.row.loanYears }}</p>
            <p>总房款: {{ Math.round(scope.row.totalAmount / 10000) }} 万</p>
            <p>首付: {{ scope.row.downPayment }} 万</p>
            <p>公积金贷款: {{ scope.row.gjjAmount }} 万</p>
            <p>商业贷款: {{ scope.row.businessAmount }} 万</p>
            <p>月供: {{ scope.row.monthlyPayment }} 元</p>
            <p>月收入: {{ scope.row.monthlyIncome }} 元</p>
            <p>每月剩余: {{ scope.row.monthlyBalance }} 元</p>
            <p>月供比例:{{ Math.round(scope.row.monthlyPayment / scope.row.monthlyIncome * 100) }}% </p>
          </div>


          <el-button type="danger" size="small" @click="deleteRecord(scope.$index)">删除</el-button>
        </template>
      </el-table-column>


    </el-table>

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



  .unit {
    margin: 0 5px;
    font-size: 14px;
  }

  .result {
    border-bottom: 1px solid #e7dbdb;
    align-items: center;
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
  margin-top: 10px;
}

.card-header {
  font-weight: bold;
}
</style>
