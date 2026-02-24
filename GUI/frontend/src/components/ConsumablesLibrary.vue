<script setup>
import { ref, reactive, onMounted } from 'vue'
import { MessagePlugin } from 'tdesign-vue-next'

// 耗材列表数据
const filaments = ref([])
// 加载状态
const loading = ref(false)
// 对话框状态
const isAddDialogVisible = ref(false)
const isEditDialogVisible = ref(false)
const isDeleteDialogVisible = ref(false)
// 当前操作的耗材
const currentFilament = ref(null)
// 表单数据
const formData = reactive({
  Name: '',
  Type: 'PLA',
  FILAMENT_K: [0, 0, 0],
  FILAMENT_S: [0, 0, 0]
})

// 表格列配置
const columns = [
  { title: '名称', dataKey: 'Name', width: 150, align: 'left' },
  { title: '类型', dataKey: 'Type', width: 100, align: 'left' },
  { 
    title: 'FILAMENT_K', 
    dataKey: 'FILAMENT_K', 
    width: 200,
    align: 'left',
    ellipsis: true
  },
  { 
    title: 'FILAMENT_S', 
    dataKey: 'FILAMENT_S', 
    width: 200,
    align: 'left',
    ellipsis: true
  },
  { 
    title: '操作', 
    dataKey: 'actions', 
    width: 180,
    align: 'left',
    fixed: 'right'
  }
]

// 获取耗材列表
const fetchFilaments = async () => {
  loading.value = true
  try {
    const response = await fetch('http://localhost:5000/filaments')
    const data = await response.json()
    let filamentList = []
    
    if (data.filaments) {
      filamentList = data.filaments
    } else if (data.error) {
      alert(`获取耗材失败: ${data.error || '未知错误'}`)
    } else {
      filamentList = []
    }
    
    // 处理数据，确保FILAMENT_K和FILAMENT_S被转换为字符串
    filaments.value = filamentList.map(filament => {
      return {
        ...filament,
        FILAMENT_K: JSON.stringify(filament.FILAMENT_K),
        FILAMENT_S: JSON.stringify(filament.FILAMENT_S)
      }
    })
  } catch (error) {
    alert(`获取耗材列表失败: ${error.message}`)
  } finally {
    loading.value = false
  }
}

// 打开新增对话框
const openAddDialog = () => {
  // 重置表单
  formData.Name = ''
  formData.Type = 'PLA'
  formData.FILAMENT_K = [0, 0, 0]
  formData.FILAMENT_S = [0, 0, 0]
  isAddDialogVisible.value = true
}

// 打开修改对话框
const openEditDialog = (filament) => {
  // 填充表单数据
  currentFilament.value = filament
  formData.Name = filament.Name
  formData.Type = filament.Type
  formData.FILAMENT_K = [...filament.FILAMENT_K]
  formData.FILAMENT_S = [...filament.FILAMENT_S]
  isEditDialogVisible.value = true
}

// 打开删除对话框
const openDeleteDialog = (filament) => {
  currentFilament.value = filament
  isDeleteDialogVisible.value = true
}

// 关闭所有对话框
const closeAllDialogs = () => {
  isAddDialogVisible.value = false
  isEditDialogVisible.value = false
  isDeleteDialogVisible.value = false
  currentFilament.value = null
}

// 检查名称唯一性
const checkNameUnique = async (name) => {
  try {
    const response = await fetch(`http://localhost:5000/filaments/check_name/${encodeURIComponent(name)}`)
    const data = await response.json()
    return data.success && data.is_unique
  } catch (error) {
    console.error('检查名称唯一性失败:', error)
    return false
  }
}

// 新增耗材
const addFilament = async () => {
  // 简单验证
  if (!formData.Name.trim()) {
    alert('请输入耗材名称')
    return
  }
  
  // 检查名称唯一性
  const isUnique = await checkNameUnique(formData.Name)
  if (!isUnique) {
    alert('耗材名称已存在')
    return
  }
  
  try {
    const response = await fetch('http://localhost:5000/filaments', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(formData)
    })
    
    const data = await response.json()
    if (data.success) {
      // 关闭对话框
      closeAllDialogs()
      // 刷新列表
      fetchFilaments()
    } else {
      throw new Error(data.error || '新增失败')
    }
  } catch (error) {
    console.error('新增耗材失败:', error)
    alert(`新增失败: ${error.message}`)
  }
}

// 修改耗材
const updateFilament = async () => {
  // 简单验证
  if (!formData.Name.trim()) {
    alert('请输入耗材名称')
    return
  }
  
  // 检查名称唯一性（如果名称改变）
  if (formData.Name !== currentFilament.value.Name) {
    const isUnique = await checkNameUnique(formData.Name)
    if (!isUnique) {
      alert('耗材名称已存在')
      return
    }
  }
  
  try {
    const response = await fetch(`http://localhost:5000/filaments/${encodeURIComponent(currentFilament.value.Name)}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(formData)
    })
    
    const data = await response.json()
    if (data.success) {
      // 关闭对话框
      closeAllDialogs()
      // 刷新列表
      fetchFilaments()
    } else {
      throw new Error(data.error || '修改失败')
    }
  } catch (error) {
    console.error('修改耗材失败:', error)
    alert(`修改失败: ${error.message}`)
  }
}

// 删除耗材
const deleteFilament = async () => {
  try {
    const response = await fetch(`http://localhost:5000/filaments/${encodeURIComponent(currentFilament.value.Name)}`, {
      method: 'DELETE'
    })
    
    const data = await response.json()
    if (data.success) {
      // 关闭对话框
      closeAllDialogs()
      // 刷新列表
      fetchFilaments()
    } else {
      throw new Error(data.error || '删除失败')
    }
  } catch (error) {
    console.error('删除耗材失败:', error)
    alert(`删除失败: ${error.message}`)
  }
}

// 组件挂载时获取耗材列表
onMounted(() => {
  fetchFilaments()
})
</script>

<template>
  <div class="consumables-library-container">
    <div class="page-header">
      <h1>耗材库</h1>
      <p class="page-description">管理3D打印耗材，添加、修改和删除耗材信息</p>
    </div>

    <!-- 主内容区域 -->
    <div class="content-single">
      <div class="main-panel">
        <!-- 耗材库操作区域 -->
        <div class="card">
          <div class="card-header">
            <h2 class="card-title">耗材管理</h2>
            <p class="card-description">添加、修改和删除3D打印耗材</p>
          </div>
          <div class="card-body">
            <div class="card-actions">
              <t-button type="primary" @click="openAddDialog">添加耗材</t-button>
              <t-button @click="fetchFilaments" :loading="loading">刷新列表</t-button>
            </div>
            
            <!-- 耗材列表表格 -->
            <div>
              <div v-if="loading">加载中...</div>
              <div v-else>
                <div class="simple-table-container">
                  <table class="simple-table">
                    <thead>
                      <tr>
                        <th>名称</th>
                        <th>类型</th>
                        <th>FILAMENT_K</th>
                        <th>FILAMENT_S</th>
                        <th>操作</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="(filament, index) in filaments" :key="index">
                        <td>{{ filament.Name }}</td>
                        <td>{{ filament.Type }}</td>
                        <td>{{ filament.FILAMENT_K }}</td>
                        <td>{{ filament.FILAMENT_S }}</td>
                        <td>
                          <div class="action-buttons">
                            <t-button 
                              size="small" 
                              type="primary" 
                              theme="outline" 
                              @click="openEditDialog(filament)"
                            >
                              修改
                            </t-button>
                            <t-button 
                              size="small" 
                              type="danger" 
                              theme="outline" 
                              @click="openDeleteDialog(filament)"
                            >
                              删除
                            </t-button>
                          </div>
                        </td>
                      </tr>
                      <tr v-if="filaments.length === 0">
                        <td colspan="5" style="text-align: center;">耗材列表为空</td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 新增/修改对话框 -->
    <div v-if="isAddDialogVisible || isEditDialogVisible" class="dialog-overlay">
      <div class="dialog">
        <div class="dialog-header">
          <h3>{{ isAddDialogVisible ? '新增耗材' : '修改耗材' }}</h3>
          <button class="close-btn" @click="closeAllDialogs">×</button>
        </div>
        <div class="dialog-content">
          <form class="simple-form">
            <div class="form-item">
              <label>耗材名称</label>
              <input 
                type="text" 
                v-model="formData.Name" 
                placeholder="请输入耗材名称"
                maxlength="50"
              >
            </div>
            <div class="form-item">
              <label>耗材类型</label>
              <select v-model="formData.Type">
                <option value="PLA">PLA</option>
                <option value="PETG">PETG</option>
              </select>
            </div>
            <div class="form-item">
              <label>FILAMENT_K</label>
              <div class="array-inputs">
                <input 
                  type="number" 
                  step="0.0001" 
                  v-model.number="formData.FILAMENT_K[0]" 
                  placeholder="K1"
                >
                <input 
                  type="number" 
                  step="0.0001" 
                  v-model.number="formData.FILAMENT_K[1]" 
                  placeholder="K2"
                >
                <input 
                  type="number" 
                  step="0.0001" 
                  v-model.number="formData.FILAMENT_K[2]" 
                  placeholder="K3"
                >
              </div>
            </div>
            <div class="form-item">
              <label>FILAMENT_S</label>
              <div class="array-inputs">
                <input 
                  type="number" 
                  step="0.0001" 
                  v-model.number="formData.FILAMENT_S[0]" 
                  placeholder="S1"
                >
                <input 
                  type="number" 
                  step="0.0001" 
                  v-model.number="formData.FILAMENT_S[1]" 
                  placeholder="S2"
                >
                <input 
                  type="number" 
                  step="0.0001" 
                  v-model.number="formData.FILAMENT_S[2]" 
                  placeholder="S3"
                >
              </div>
            </div>
          </form>
        </div>
        <div class="dialog-footer">
          <button class="cancel-btn" @click="closeAllDialogs">取消</button>
          <button 
            class="confirm-btn" 
            @click="isAddDialogVisible ? addFilament() : updateFilament()"
          >
            确认
          </button>
        </div>
      </div>
    </div>
    
    <!-- 删除确认对话框 -->
    <div v-if="isDeleteDialogVisible" class="dialog-overlay">
      <div class="dialog">
        <div class="dialog-header">
          <h3>删除确认</h3>
          <button class="close-btn" @click="closeAllDialogs">×</button>
        </div>
        <div class="dialog-content">
          <p>确定要删除耗材 <strong>{{ currentFilament?.Name }}</strong> 吗？</p>
          <p>此操作不可恢复，请谨慎执行。</p>
        </div>
        <div class="dialog-footer">
          <button class="cancel-btn" @click="closeAllDialogs">取消</button>
          <button class="delete-confirm-btn" @click="deleteFilament()">确认删除</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 全局样式 - 匹配模型生成页面 */
.consumables-library-container {
  padding: 24px;
  max-width: 1200px;
  margin: 0 auto;
}

/* 页面头部 - 匹配模型生成页面 */
.page-header {
  margin-bottom: 32px;
  text-align: center;
}

.page-header h1 {
  font-size: 28px;
  font-weight: 700;
  color: #1a1a1a;
  margin-bottom: 8px;
}

.page-description {
  font-size: 16px;
  color: #666;
  margin: 0;
}

/* 单一列布局 - 匹配模型生成页面 */
.content-single {
  display: flex;
  justify-content: center;
}

/* 主面板样式 - 匹配模型生成页面 */
.main-panel {
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
  max-width: 800px;
}

/* 卡片样式 - 匹配模型生成页面 */
.card {
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  transition: all 0.3s ease;
}

.card:hover {
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.12);
  transform: translateY(-2px);
}

.card-header {
  padding: 20px 24px;
  border-bottom: 1px solid #f0f0f0;
}

.card-title {
  font-size: 18px;
  font-weight: 600;
  color: #1a1a1a;
  margin: 0 0 4px 0;
}

.card-description {
  font-size: 14px;
  color: #666;
  margin: 0;
}

.card-body {
  padding: 24px;
}

/* 表格容器 - 匹配模型生成页面 */
.table-container {
  margin-top: 20px;
  overflow-x: auto;
  border-radius: 8px;
  background: #fafafa;
  padding: 20px;
  border: 1px solid #f0f0f0;
}

/* 表格样式 - 匹配模型生成页面 */
.simple-table {
  width: 100%;
  border-collapse: collapse;
  background: #ffffff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.simple-table th,
.simple-table td {
  padding: 12px 16px;
  text-align: left;
  border-bottom: 1px solid #f0f0f0;
}

.simple-table th {
  background-color: #fafafa;
  font-weight: 600;
  color: #333;
  font-size: 14px;
  white-space: nowrap;
}

.simple-table tr:hover {
  background-color: #fafafa;
}

/* 操作按钮样式 - 匹配模型生成页面 */
.action-buttons {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.action-buttons :deep(.t-button) {
  margin-bottom: 4px;
  padding: 4px 12px;
  font-size: 13px;
}

/* 卡片操作区 - 匹配模型生成页面 */
.card-actions {
  margin-bottom: 20px;
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

/* 对话框样式 - 匹配模型生成页面 */
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.dialog {
  background-color: white;
  border-radius: 12px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.15);
  width: 700px;
  max-width: 90%;
  max-height: 90vh;
  overflow-y: auto;
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  border-bottom: 1px solid #f0f0f0;
}

.dialog-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #1a1a1a;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #666;
  padding: 0;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: all 0.3s;
}

.close-btn:hover {
  background-color: #f0f0f0;
  color: #333;
}

.dialog-content {
  padding: 24px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 20px 24px;
  border-top: 1px solid #f0f0f0;
}

/* 表单样式 - 匹配模型生成页面 */
.simple-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-item label {
  font-weight: 500;
  font-size: 14px;
  color: #333;
  display: flex;
  align-items: center;
  gap: 6px;
}

.form-item input,
.form-item select {
  padding: 10px 16px;
  border: 1px solid #dcdfe6;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.2s ease;
  width: 100%;
}

.form-item input:focus,
.form-item select:focus {
  outline: none;
  border-color: #409eff;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
}

.array-inputs {
  display: flex;
  gap: 12px;
}

.array-inputs input {
  flex: 1;
}

/* 按钮样式 - 匹配模型生成页面 */
.cancel-btn,
.confirm-btn,
.delete-confirm-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s;
  min-width: 80px;
}

.cancel-btn {
  background-color: #f5f7fa;
  color: #333;
  border: 1px solid #e4e7ed;
}

.cancel-btn:hover {
  background-color: #e4e7ed;
}

.confirm-btn {
  background-color: #409eff;
  color: white;
}

.confirm-btn:hover {
  background-color: #66b1ff;
}

.delete-confirm-btn {
  background-color: #f56c6c;
  color: white;
}

.delete-confirm-btn:hover {
  background-color: #f78989;
}

/* 响应式设计 - 匹配模型生成页面 */
@media (max-width: 992px) {
  .main-panel {
    max-width: 100%;
  }
}

@media (max-width: 768px) {
  .consumables-library-container {
    padding: 16px;
  }
  
  .page-header h1 {
    font-size: 24px;
  }
  
  .card-header,
  .card-body {
    padding: 16px;
  }
  
  .dialog {
    width: 95%;
  }
  
  .dialog-header,
  .dialog-content,
  .dialog-footer {
    padding: 16px;
  }
}
</style>