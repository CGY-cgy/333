<!--
会员管理组件
API配置信息：
- 开发环境：http://localhost:8080/api/v1
- 生产环境：https://api.lingjing.com/v1
- 请求超时时间：30秒
对应API接口：
- GET /api/members - 获取会员列表
- GET /api/members/statistics - 获取会员统计数据
- GET /api/points - 获取积分记录
- POST /api/points - 增加积分
- GET /api/levels - 获取等级列表
- POST /api/levels - 创建等级
- PUT /api/levels/:id - 更新等级
- DELETE /api/levels/:id - 删除等级
- GET /api/members/report - 获取会员报表
-->
<template>
  <div class="membership-container">
    <!-- 页面标题和操作 -->
    <div class="header-section">
      <h1 class="page-title">会员管理</h1>
      <p class="page-subtitle">会员分析、积分管理与等级管理</p>
    </div>

    <!-- 标签页切换 -->
    <el-tabs v-model="activeTab" class="main-tabs">
      <!-- 会员分析 -->
      <el-tab-pane label="会员分析" name="analysis">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <div class="date-range">
            <el-select v-model="currentDateRange" placeholder="选择日期范围" size="large" @change="updateAllCharts">
              <el-option label="最近7天" :value="7" />
              <el-option label="最近30天" :value="30" />
              <el-option label="最近90天" :value="90" />
            </el-select>
          </div>
          <el-button type="primary" size="large" @click="exportReport">
            <el-icon><el-icon-document-download /></el-icon>
            <span>导出报表</span>
          </el-button>
        </div>

        <!-- 统计数据网格 -->
        <div class="stats-grid">
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon revenue">💰</div>
              <span class="stat-trend up">↑ 12.5%</span>
            </div>
            <div class="stat-value">¥{{ formatNumber(486920) }}</div>
            <div class="stat-label">总收入</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon members">💎</div>
              <span class="stat-trend up">↑ 8.3%</span>
            </div>
            <div class="stat-value">{{ formatNumber(12580) }}</div>
            <div class="stat-label">付费会员数</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon conversion">🎯</div>
              <span class="stat-trend up">↑ 3.2%</span>
            </div>
            <div class="stat-value">23.4%</div>
            <div class="stat-label">付费转化率</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon arpu">📊</div>
              <span class="stat-trend up">↑ 5.7%</span>
            </div>
            <div class="stat-value">¥{{ formatNumber(387) }}</div>
            <div class="stat-label">ARPU</div>
          </el-card>
        </div>
      </el-tab-pane>

      <!-- 积分管理 -->
      <el-tab-pane label="积分管理" name="points">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="openAddPointsDialog">
            <el-icon><el-icon-plus /></el-icon>
            <span>批量加分</span>
          </el-button>
          <el-button @click="exportPointsReport">
            <el-icon><el-icon-document-download /></el-icon>
            <span>导出积分报表</span>
          </el-button>
        </div>

        <!-- 搜索和筛选 -->
        <div class="search-filter">
          <el-input v-model="pointsSearchForm.userId" placeholder="会员ID" clearable class="search-input">
            <template #prefix>
              <el-icon><el-icon-search /></el-icon>
            </template>
          </el-input>
          <el-select v-model="pointsSearchForm.type" placeholder="变动类型" clearable>
            <el-option label="增加" value="increase" />
            <el-option label="减少" value="decrease" />
          </el-select>
        </div>
      </el-tab-pane>

      <!-- 等级管理 -->
      <el-tab-pane label="等级管理" name="levels">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="openAddLevelDialog">
            <el-icon><el-icon-plus /></el-icon>
            <span>添加等级</span>
          </el-button>
          <el-button @click="exportLevelsReport">
            <el-icon><el-icon-document-download /></el-icon>
            <span>导出等级报表</span>
          </el-button>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import {
  Download,
  Plus,
  Search
} from '@element-plus/icons-vue'

// 定义组件别名
const ElIconDocumentDownload = Download
const ElIconPlus = Plus
const ElIconSearch = Search

// 响应式数据
const currentDateRange = ref(30)
const activeTab = ref('analysis')

// 格式化数字
const formatNumber = (num) => {
  return num.toLocaleString()
}

// 更新所有图表
const updateAllCharts = () => {
  ElMessage.success('图表数据已更新')
}

// 导出报表
const exportReport = () => {
  ElMessage.success('报表导出成功')
}

// 打开加分对话框
const openAddPointsDialog = () => {
  ElMessage.info('打开加分对话框')
}

// 导出积分报表
const exportPointsReport = () => {
  ElMessage.success('积分报表导出成功')
}

// 打开添加等级对话框
const openAddLevelDialog = () => {
  ElMessage.info('打开添加等级对话框')
}

// 导出等级报表
const exportLevelsReport = () => {
  ElMessage.success('等级报表导出成功')
}

// 积分搜索表单
const pointsSearchForm = ref({
  userId: '',
  type: ''
})
</script>

<style scoped>
.membership-container {
  padding: 0;
  background-color: var(--el-bg-color);
  min-height: 100vh;
}

.header-section {
  margin-bottom: 20px;
  padding: 20px 0;
}

.page-title {
  font-size: 24px;
  font-weight: 700;
  color: var(--el-text-color-primary);
  margin: 0;
}

.page-subtitle {
  font-size: 14px;
  color: var(--el-text-color-secondary);
  margin: 4px 0 0 0;
}

.main-tabs {
  margin-bottom: 20px;
}

.top-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  margin-bottom: 20px;
  padding: 16px;
  background-color: var(--el-bg-color-page);
  border-radius: var(--el-border-radius-base);
  box-shadow: var(--el-box-shadow-light);
}

.top-actions .el-button {
  font-weight: 500;
  transition: all 0.3s ease;
}

.top-actions .el-button:hover {
  transform: translateY(-2px);
  box-shadow: var(--el-box-shadow-light);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.stat-card {
  padding: 20px;
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
  border-radius: var(--el-border-radius-large);
  transition: all 0.3s ease;
}

.stat-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 20px rgba(0, 0, 0, 0.1);
}

.stat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.stat-icon {
  font-size: 24px;
  font-weight: bold;
}

.stat-icon.revenue {
  color: var(--el-color-primary);
}

.stat-icon.members {
  color: var(--el-color-success);
}

.stat-icon.conversion {
  color: var(--el-color-warning);
}

.stat-icon.arpu {
  color: var(--el-color-danger);
}

.stat-trend {
  font-size: 12px;
  font-weight: 500;
  padding: 4px 8px;
  border-radius: 12px;
}

.stat-trend.up {
  background-color: rgba(72, 187, 120, 0.1);
  color: var(--el-color-success);
}

.stat-trend.down {
  background-color: rgba(245, 101, 101, 0.1);
  color: var(--el-color-danger);
}

.stat-value {
  font-size: 32px;
  font-weight: 700;
  color: var(--el-text-color-primary);
  margin-bottom: 4px;
}

.stat-label {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.search-filter {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  margin-bottom: 20px;
  padding: 16px;
  background-color: var(--el-bg-color-page);
  border-radius: var(--el-border-radius-base);
  box-shadow: var(--el-box-shadow-light);
}

.search-input {
  width: 240px;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .top-actions {
    flex-direction: column;
    align-items: stretch;
  }
  
  .stats-grid {
    grid-template-columns: 1fr;
  }
  
  .search-filter {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>