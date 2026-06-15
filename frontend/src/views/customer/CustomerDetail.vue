<template>
  <div class="customer-detail-container" v-loading="pageLoading">
    <div class="detail-header">
      <div class="header-left">
        <el-button @click="router.push('/customer')">
          <el-icon><ArrowLeft /></el-icon>返回
        </el-button>
        <el-avatar :size="64" :src="customer?.avatar" class="customer-avatar">
          <el-icon :size="32"><User /></el-icon>
        </el-avatar>
        <div class="customer-summary">
          <div class="customer-name-row">
            <span class="customer-name">{{ customer?.name || '' }}</span>
            <el-tag :type="customerTypeTagType(customer?.customerType)" size="default">
              {{ customerTypeLabel(customer?.customerType) }}
            </el-tag>
            <el-icon v-if="customer?.importance === 3" class="crown-icon" :size="22" color="#e6a23c">
              <GoldMedal />
            </el-icon>
            <el-tag :type="statusTagType(customer?.status)" size="default">
              {{ statusLabel(customer?.status) }}
            </el-tag>
          </div>
        </div>
      </div>
      <div class="header-actions">
        <el-button v-if="hasPermission('customer:edit')" type="primary" @click="handleEdit">
          <el-icon><Edit /></el-icon>编辑
        </el-button>
        <el-button
          v-if="customer?.status === 1 && hasPermission('customer:freeze')"
          type="warning"
          @click="openFreezeDialog"
        >
          <el-icon><Lock /></el-icon>冻结
        </el-button>
        <el-button
          v-if="customer?.status === 2 && hasPermission('customer:unfreeze')"
          type="success"
          @click="openUnfreezeDialog"
        >
          <el-icon><Unlock /></el-icon>解冻
        </el-button>
        <el-button v-if="hasPermission('customer:delete')" type="danger" @click="handleDelete">
          <el-icon><Delete /></el-icon>删除
        </el-button>
      </div>
    </div>

    <el-row :gutter="16" class="stats-row">
      <el-col :span="6">
        <el-card shadow="never" class="stat-card">
          <div class="stat-value">{{ customer?.totalOrders ?? 0 }}</div>
          <div class="stat-label">总订单数</div>
        </el-card>
      </el-col>
      <el-col v-if="hasPermission('customer:finance:view')" :span="6">
        <el-card shadow="never" class="stat-card">
          <div class="stat-value">¥{{ customer?.totalSpent ?? 0 }}</div>
          <div class="stat-label">总消费额</div>
        </el-card>
      </el-col>
      <el-col v-if="hasPermission('customer:finance:view')" :span="6">
        <el-card shadow="never" class="stat-card">
          <div class="stat-value">¥{{ customer?.avgSpent ?? 0 }}</div>
          <div class="stat-label">平均消费</div>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card shadow="never" class="stat-card">
          <div class="stat-value">{{ customer?.lifecycleDays ?? 0 }}天</div>
          <div class="stat-label">客户生命周期</div>
        </el-card>
      </el-col>
    </el-row>

    <el-card shadow="never" class="tabs-card">
      <el-tabs v-model="activeTab">
        <el-tab-pane label="基本信息" name="basic">
          <div v-if="customer" class="info-sections">
            <div class="info-section">
              <h4 class="section-title">个人信息</h4>
              <el-descriptions :column="2" border>
                <el-descriptions-item label="姓名">{{ customer.name }}</el-descriptions-item>
                <el-descriptions-item label="性别">{{ genderLabel(customer.gender) }}</el-descriptions-item>
                <el-descriptions-item label="客户类型">{{ customerTypeLabel(customer.customerType) }}</el-descriptions-item>
                <el-descriptions-item label="客户来源">{{ customerSourceLabel(customer.customerSource) }}</el-descriptions-item>
                <el-descriptions-item label="重要程度">{{ importanceLabel(customer.importance) }}</el-descriptions-item>
                <el-descriptions-item label="状态">{{ statusLabel(customer.status) }}</el-descriptions-item>
                <el-descriptions-item label="生日">{{ customer.birthday || '-' }}</el-descriptions-item>
                <el-descriptions-item label="备注" :span="2">{{ customer.remark || '-' }}</el-descriptions-item>
              </el-descriptions>
            </div>

            <div class="info-section">
              <h4 class="section-title">证件信息</h4>
              <el-descriptions :column="2" border>
                <el-descriptions-item label="证件类型">{{ idTypeLabel(customer.idType) }}</el-descriptions-item>
                <el-descriptions-item label="证件号码">{{ customer.idNumber || '-' }}</el-descriptions-item>
              </el-descriptions>
            </div>

            <div class="info-section">
              <h4 class="section-title">联系信息</h4>
              <el-descriptions :column="2" border>
                <el-descriptions-item label="手机号">{{ customer.phone || '-' }}</el-descriptions-item>
                <el-descriptions-item label="邮箱">{{ customer.email || '-' }}</el-descriptions-item>
                <el-descriptions-item label="微信号">{{ customer.wechat || '-' }}</el-descriptions-item>
                <el-descriptions-item label="QQ号">{{ customer.qq || '-' }}</el-descriptions-item>
              </el-descriptions>
            </div>

            <div class="info-section">
              <h4 class="section-title">地址信息</h4>
              <div v-if="addresses.length" class="address-list">
                <div v-for="addr in addresses" :key="addr.id" class="address-card">
                  <div class="address-card-body">
                    <div class="address-header">
                      <el-tag :type="addressTypeTagType(addr.addressType)" size="small">
                        {{ addressTypeLabel(addr.addressType) }}
                      </el-tag>
                      <el-tag v-if="addr.isDefault" type="success" size="small" class="default-tag">默认</el-tag>
                    </div>
                    <div class="address-detail">{{ addr.fullAddress || addr.address }}</div>
                    <div v-if="addr.postalCode" class="address-postal">邮编：{{ addr.postalCode }}</div>
                  </div>
                  <div class="address-card-actions">
                    <el-button type="primary" link size="small" @click="handleEditAddress(addr)">
                      <el-icon><Edit /></el-icon>编辑
                    </el-button>
                    <el-button type="danger" link size="small" @click="handleDeleteAddress(addr.id)">
                      <el-icon><Delete /></el-icon>删除
                    </el-button>
                  </div>
                </div>
              </div>
              <el-empty v-else description="暂无地址信息" :image-size="60" />
            </div>
          </div>
        </el-tab-pane>

        <el-tab-pane label="入住记录" name="stay">
          <el-empty description="暂无数据，客户尚未完成入住" :image-size="100" />
        </el-tab-pane>

        <el-tab-pane label="消费记录" name="payment">
          <el-empty description="暂无数据，客户尚未产生消费" :image-size="100" />
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <el-dialog
      v-model="freezeDialogVisible"
      :title="freezeDialogType === 'freeze' ? '冻结客户' : '解冻客户'"
      width="480px"
      destroy-on-close
    >
      <el-alert
        v-if="freezeDialogType === 'freeze'"
        title="冻结后该客户将无法预订和入住"
        type="warning"
        :closable="false"
        show-icon
        class="freeze-warning"
      />
      <el-form ref="freezeFormRef" :model="freezeForm" :rules="freezeRules" label-width="80px">
        <el-form-item label="原因" prop="reason">
          <el-input
            v-model="freezeForm.reason"
            type="textarea"
            :rows="4"
            placeholder="请输入原因"
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="freezeDialogVisible = false">取消</el-button>
        <el-button type="primary" :loading="freezeSaving" @click="handleFreezeSubmit">确认</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import { ArrowLeft, User, Edit, Delete, Lock, Unlock, GoldMedal } from '@element-plus/icons-vue'
import { useUserStore } from '@/stores/user'
import api from '@/api'

const route = useRoute()
const router = useRouter()
const userStore = useUserStore()
const id = route.params.id
const hasPermission = (p) => userStore.hasPermission(p)

const genderLabel = (v) => {
  const map = { 0: '未知', 1: '男', 2: '女' }
  return map[v] ?? '-'
}

const customerTypeLabel = (v) => {
  const map = { 1: '散客', 2: '企业客户', 3: '协议客户', 4: 'VIP客户' }
  return map[v] ?? '-'
}

const customerTypeTagType = (v) => {
  const map = { 1: 'info', 2: '', 3: 'warning', 4: 'danger' }
  return map[v] ?? 'info'
}

const customerSourceLabel = (v) => {
  const map = { 1: '官网', 2: 'OTA平台', 3: '企业协议', 4: '电话预订', 5: '前台登记', 6: '老客户推荐' }
  return map[v] ?? '-'
}

const importanceLabel = (v) => {
  const map = { 1: '普通', 2: '重要', 3: 'VIP' }
  return map[v] ?? '-'
}

const statusLabel = (v) => {
  const map = { 1: '正常', 2: '冻结' }
  return map[v] ?? '-'
}

const statusTagType = (v) => {
  const map = { 1: 'success', 2: 'danger' }
  return map[v] ?? 'info'
}

const idTypeLabel = (v) => {
  const map = { 1: '身份证', 2: '护照', 3: '港澳通行证', 4: '台胞证', 5: '驾驶证' }
  return map[v] ?? '-'
}

const addressTypeLabel = (v) => {
  const map = { 1: '家庭地址', 2: '公司地址', 3: '发票邮寄地址' }
  return map[v] ?? '-'
}

const addressTypeTagType = (v) => {
  const map = { 1: '', 2: 'warning', 3: 'info' }
  return map[v] ?? 'info'
}

const activeTab = ref('basic')
const customer = ref(null)
const addresses = ref([])
const pageLoading = ref(false)

const freezeDialogVisible = ref(false)
const freezeDialogType = ref('freeze')
const freezeSaving = ref(false)
const freezeFormRef = ref(null)
const freezeForm = reactive({ reason: '' })
const freezeRules = {
  reason: [{ required: true, message: '请输入原因', trigger: 'blur' }]
}

const loadCustomer = async () => {
  pageLoading.value = true
  try {
    const res = await api.customer.getById(id)
    if (res.code === 200 && res.data) {
      customer.value = res.data
    } else {
      ElMessage.error(res.message || '获取客户详情失败')
    }
  } catch {
    ElMessage.error('获取客户详情失败')
  } finally {
    pageLoading.value = false
  }
}

const loadAddresses = async () => {
  try {
    const res = await api.customer.getAddresses(id)
    if (res.code === 200) {
      addresses.value = res.data || []
    }
  } catch {
    addresses.value = []
  }
}

const handleEdit = () => {
  router.push(`/customer/edit/${id}`)
}

const openFreezeDialog = () => {
  freezeDialogType.value = 'freeze'
  freezeForm.reason = ''
  freezeDialogVisible.value = true
}

const openUnfreezeDialog = () => {
  freezeDialogType.value = 'unfreeze'
  freezeForm.reason = ''
  freezeDialogVisible.value = true
}

const handleFreezeSubmit = async () => {
  const valid = await freezeFormRef.value.validate().catch(() => false)
  if (!valid) return

  freezeSaving.value = true
  try {
    const action = freezeDialogType.value === 'freeze' ? api.customer.freeze : api.customer.unfreeze
    const res = await action(id, { reason: freezeForm.reason })
    if (res.code === 200) {
      ElMessage.success(freezeDialogType.value === 'freeze' ? '冻结成功' : '解冻成功')
      freezeDialogVisible.value = false
      await loadCustomer()
    } else {
      ElMessage.error(res.message || '操作失败')
    }
  } catch {
    ElMessage.error('操作失败')
  } finally {
    freezeSaving.value = false
  }
}

const handleDelete = async () => {
  try {
    await ElMessageBox.confirm(`确认删除客户「${customer.value?.name}」？删除后不可恢复！`, '警告', {
      confirmButtonText: '确认删除',
      cancelButtonText: '取消',
      type: 'warning'
    })
    const res = await api.customer.delete(id)
    if (res.code === 200) {
      ElMessage.success('删除成功')
      router.push('/customer')
    } else {
      ElMessage.error(res.message || '删除失败')
    }
  } catch {}
}

const handleEditAddress = (addr) => {
  router.push({ path: `/customer/edit/${id}`, query: { editAddress: addr.id } })
}

const handleDeleteAddress = async (addrId) => {
  try {
    await ElMessageBox.confirm('确认删除该地址？', '警告', {
      confirmButtonText: '确认删除',
      cancelButtonText: '取消',
      type: 'warning'
    })
    const res = await api.customer.deleteAddress(addrId)
    if (res.code === 200) {
      ElMessage.success('删除成功')
      await loadAddresses()
    } else {
      ElMessage.error(res.message || '删除失败')
    }
  } catch {}
}

onMounted(() => {
  loadCustomer()
  loadAddresses()
})
</script>

<style scoped>
.customer-detail-container {
  padding: 10px;
}

.detail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding: 16px 20px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.06);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 16px;
}

.customer-avatar {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  flex-shrink: 0;
}

.customer-summary {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.customer-name-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.customer-name {
  font-size: 24px;
  font-weight: 700;
  color: #1a202c;
}

.crown-icon {
  flex-shrink: 0;
}

.header-actions {
  display: flex;
  gap: 10px;
}

.stats-row {
  margin-bottom: 16px;
}

.stat-card {
  border-radius: 12px;
  border: none;
  text-align: center;
}

.stat-value {
  font-size: 22px;
  font-weight: 700;
  color: #2d3748;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 13px;
  color: #718096;
}

.tabs-card {
  border-radius: 12px;
  border: none;
}

.info-sections {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.info-section {
  margin: 0;
}

.section-title {
  font-size: 16px;
  font-weight: 600;
  color: #1a202c;
  margin: 0 0 12px 0;
  padding-left: 10px;
  border-left: 3px solid #409eff;
}

.address-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
  gap: 12px;
}

.address-card {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 14px 16px;
  background: #f7f8fa;
  border-radius: 10px;
  border: 1px solid #ebeef5;
}

.address-card-body {
  flex: 1;
}

.address-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
}

.default-tag {
  margin-left: 0;
}

.address-detail {
  font-size: 14px;
  color: #2d3748;
  line-height: 1.6;
}

.address-postal {
  font-size: 13px;
  color: #909399;
  margin-top: 4px;
}

.address-card-actions {
  display: flex;
  flex-direction: column;
  gap: 4px;
  flex-shrink: 0;
  margin-left: 12px;
}

.freeze-warning {
  margin-bottom: 16px;
}
</style>
