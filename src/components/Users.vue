<!--
用户管理组件
API配置信息：
- 开发环境：http://localhost:8080/api/v1
- 生产环境：https://api.lingjing.com/v1
- 请求超时时间：30秒
对应API接口：
- GET /api/users - 获取用户列表
- POST /api/users - 创建用户
- GET /api/users/:id - 获取用户详情
- PUT /api/users/:id - 更新用户
- DELETE /api/users/:id - 删除用户
- GET /api/users/statistics - 获取用户统计数据
- POST /api/users/import - 导入用户
- GET /api/users/export - 导出用户
-->
<template>
  <div class="users-container">
    <!-- 页面标题和操作 -->
    <div class="header-section">
      <h1 class="page-title">用户管理</h1>
      <p class="page-subtitle">用户列表、详情、分析与批量操作</p>
    </div>

    <!-- 标签页切换 -->
    <el-tabs v-model="activeTab" class="main-tabs">
      <!-- 用户列表 -->
      <el-tab-pane label="用户列表" name="list">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="exportExcel">
            <el-icon><el-icon-document /></el-icon>
            <span>导出Excel</span>
          </el-button>
          <el-button type="success" @click="openBatchOperationDialog">
            <el-icon><el-icon-s-operation /></el-icon>
            <span>批量操作</span>
          </el-button>
        </div>

        <!-- 搜索卡片 -->
        <el-card shadow="hover" class="search-card">
          <h3 class="section-title">搜索条件</h3>
          <div class="search-form">
            <div class="form-row">
              <el-form-item label="用户ID">
                <el-input v-model="searchForm.id" placeholder="请输入用户ID" clearable />
              </el-form-item>
              <el-form-item label="手机号">
                <el-input v-model="searchForm.phone" placeholder="请输入手机号" clearable />
              </el-form-item>
              <el-form-item label="昵称">
                <el-input v-model="searchForm.name" placeholder="请输入昵称" clearable />
              </el-form-item>
              <el-form-item label="会员状态">
                <el-select v-model="searchForm.memberStatus" placeholder="请选择会员状态" clearable>
                  <el-option label="普通用户" value="normal" />
                  <el-option label="VIP会员" value="vip" />
                  <el-option label="SVIP会员" value="svip" />
                </el-select>
              </el-form-item>
              <el-form-item label="账户状态">
                <el-select v-model="searchForm.accountStatus" placeholder="请选择账户状态" clearable>
                  <el-option label="已启用" value="active" />
                  <el-option label="已禁用" value="inactive" />
                  <el-option label="已封禁" value="banned" />
                </el-select>
              </el-form-item>
            </div>
            <div class="form-actions">
              <el-button type="primary" @click="searchUsers">
                <el-icon><el-icon-search /></el-icon>
                <span>搜索</span>
              </el-button>
              <el-button @click="resetSearch">
                <el-icon><el-icon-refresh-right /></el-icon>
                <span>重置</span>
              </el-button>
            </div>
          </div>
        </el-card>

        <!-- 用户列表表格 -->
        <el-card shadow="hover" class="users-table-card">
          <h3 class="section-title">用户列表</h3>
          <div class="table-container">
            <el-table :data="currentPageUsers" stripe style="width: 100%">
              <el-table-column type="selection" width="55" />
              <el-table-column label="用户" min-width="180">
                <template #default="scope">
                  <div class="user-info">
                    <div class="user-avatar" :style="{ background: `linear-gradient(135deg, ${getUserAvatarColor(scope.row.id)} 0%, ${getUserAvatarColor(scope.row.id, true)} 100%)` }">
                      {{ scope.row.name.charAt(0) }}
                    </div>
                    <div class="user-details">
                      <div class="user-name">{{ scope.row.name }}</div>
                      <div class="user-id">ID: {{ scope.row.id }}</div>
                    </div>
                  </div>
                </template>
              </el-table-column>
              <el-table-column prop="phone" label="手机号" min-width="120" />
              <el-table-column prop="registerTime" label="注册时间" min-width="160" />
              <el-table-column prop="lastLogin" label="最后登录" min-width="160" />
              <el-table-column label="会员状态" min-width="120">
                <template #default="scope">
                  <el-tag :type="getMemberStatusType(scope.row.memberStatus)">
                    {{ getMemberStatusText(scope.row.memberStatus) }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column label="账户状态" min-width="120">
                <template #default="scope">
                  <el-tag :type="getAccountStatusType(scope.row.accountStatus)">
                    {{ getAccountStatusText(scope.row.accountStatus) }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column label="操作" min-width="150" fixed="right">
                <template #default="scope">
                  <el-button type="primary" link @click="viewUserDetail(scope.row)">
                    <el-icon><el-icon-view /></el-icon>
                    <span>详情</span>
                  </el-button>
                  <el-button type="warning" link @click="resetPassword(scope.row.id)">
                    <el-icon><el-icon-key /></el-icon>
                    <span>重置密码</span>
                  </el-button>
                  <el-button :type="scope.row.accountStatus === 'active' ? 'danger' : 'success'" link @click="toggleStatus(scope.row)">
                    <el-icon><el-icon-refresh /></el-icon>
                    <span>{{ scope.row.accountStatus === 'active' ? '禁用' : '启用' }}</span>
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </div>

          <!-- 分页 -->
          <div class="pagination-container">
            <div class="pagination-info">
              共 {{ filteredUsers.length }} 条记录，每页 {{ pageSize }} 条
            </div>
            <el-pagination
              v-model:current-page="currentPage"
              v-model:page-size="pageSize"
              :page-sizes="[5, 10, 20, 50]"
              layout="total, sizes, prev, pager, next, jumper"
              :total="filteredUsers.length"
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
            />
          </div>
        </el-card>
      </el-tab-pane>

      <!-- 用户详情 -->
      <el-tab-pane label="用户详情" name="detail">
        <el-card shadow="hover" class="detail-card" v-if="selectedMember">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">{{ selectedMember.name }}</h3>
              <el-tag :type="getAccountStatusType(selectedMember.accountStatus)">
                {{ getAccountStatusText(selectedMember.accountStatus) }}
              </el-tag>
            </div>
          </template>
          <div class="user-detail">
            <div class="detail-header">
              <div class="user-avatar large" :style="{ background: `linear-gradient(135deg, ${getUserAvatarColor(selectedMember.id)} 0%, ${getUserAvatarColor(selectedMember.id, true)} 100%)` }">
                {{ selectedMember.name.charAt(0) }}
              </div>
              <div class="user-info-detail">
                <h4 class="user-name">{{ selectedMember.name }}</h4>
                <div class="user-basic-info">
                  <span>ID: {{ selectedMember.id }}</span>
                  <span>{{ selectedMember.fullPhone }}</span>
                </div>
                <div class="member-status-info">
                  <el-tag :type="getMemberStatusType(selectedMember.memberStatus)" size="large">
                    {{ getMemberStatusText(selectedMember.memberStatus) }}
                  </el-tag>
                </div>
              </div>
            </div>

            <div class="detail-sections">
              <div class="detail-section">
                <h4 class="section-title">基本信息</h4>
                <div class="detail-grid">
                  <div class="detail-item">
                    <span class="detail-label">注册时间：</span>
                    <span class="detail-value">{{ selectedMember.registerTime }}</span>
                  </div>
                  <div class="detail-item">
                    <span class="detail-label">最后登录：</span>
                    <span class="detail-value">{{ selectedMember.lastLogin }}</span>
                  </div>
                  <div class="detail-item">
                    <span class="detail-label">会员状态：</span>
                    <span class="detail-value">{{ getMemberStatusText(selectedMember.memberStatus) }}</span>
                  </div>
                  <div class="detail-item">
                    <span class="detail-label">账户状态：</span>
                    <span class="detail-value">{{ getAccountStatusText(selectedMember.accountStatus) }}</span>
                  </div>
                </div>
              </div>

              <div class="detail-section">
                <h4 class="section-title">健康信息</h4>
                <div class="health-info">
                  <div class="health-score-card">
                    <div class="score-circle">
                      <span class="score-value">{{ selectedMember.healthScore }}</span>
                    </div>
                    <div class="score-desc">
                      <p>基于舌像和问卷的综合评估</p>
                    </div>
                  </div>
                  <div class="constitution-tags">
                    <el-tag v-for="(item, index) in selectedMember.constitution" :key="index" size="large">
                      {{ item }}
                    </el-tag>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="card-actions">
            <el-button type="primary" @click="openEditModal">修改信息</el-button>
            <el-button type="warning" @click="resetPassword(selectedMember.id)">重置密码</el-button>
            <el-button type="danger" @click="banUser(selectedMember.id)">
              {{ selectedMember.accountStatus === 'banned' ? '解除封禁' : '封禁账户' }}
            </el-button>
          </div>
        </el-card>
        <div class="empty-detail" v-else>
          <el-empty description="请选择一个用户查看详情" />
        </div>
      </el-tab-pane>

      <!-- 用户分析 -->
      <el-tab-pane label="用户分析" name="analysis">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <div class="date-range">
            <el-select v-model="currentDateRange" placeholder="选择日期范围" size="large" @change="updateAllCharts">
              <el-option label="最近7天" :value="7" />
              <el-option label="最近30天" :value="30" />
              <el-option label="最近90天" :value="90" />
            </el-select>
          </div>
          <el-button type="primary" @click="exportAnalysisReport">
            <el-icon><el-icon-document-download /></el-icon>
            <span>导出分析报表</span>
          </el-button>
        </div>

        <!-- 统计数据网格 -->
        <div class="stats-grid">
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon users">👥</div>
              <span class="stat-trend up">↑ 12.5%</span>
            </div>
            <div class="stat-value">{{ formatNumber(12580) }}</div>
            <div class="stat-label">总用户数</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon new-users">🚀</div>
              <span class="stat-trend up">↑ 8.3%</span>
            </div>
            <div class="stat-value">{{ formatNumber(1250) }}</div>
            <div class="stat-label">本月新增</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon active">💡</div>
              <span class="stat-trend up">↑ 3.2%</span>
            </div>
            <div class="stat-value">{{ formatNumber(8920) }}</div>
            <div class="stat-label">活跃用户</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon members">💎</div>
              <span class="stat-trend up">↑ 5.7%</span>
            </div>
            <div class="stat-value">{{ formatNumber(3250) }}</div>
            <div class="stat-label">付费会员</div>
          </el-card>
        </div>

        <!-- 用户增长趋势与会员分布 -->
        <div class="grid-2">
          <!-- 用户增长趋势 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户增长趋势</h3>
                <span class="chart-desc">各周期用户增长数据</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="growthTrendChart"></canvas>
            </div>
          </el-card>

          <!-- 会员分布 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">会员分布</h3>
                <span class="chart-desc">不同会员等级的分布情况</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="memberDistributionChart"></canvas>
            </div>
            <div class="member-distribution-info">
              <div class="member-item" v-for="item in memberDistributionData" :key="item.level">
                <div class="member-name">{{ item.name }}</div>
                <div class="member-stats">
                  <div class="stat">
                    <span class="stat-label">用户数:</span>
                    <span class="stat-value">{{ formatNumber(item.count) }}</span>
                  </div>
                  <div class="stat">
                    <span class="stat-label">占比:</span>
                    <span class="stat-value">{{ item.percentage }}%</span>
                  </div>
                </div>
              </div>
            </div>
          </el-card>
        </div>

        <!-- 用户活跃度分析 -->
        <el-card shadow="hover" class="chart-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">用户活跃度分析</h3>
              <span class="chart-desc">用户登录和操作活跃度</span>
            </div>
          </template>
          <div class="chart-container">
            <canvas ref="activityChart"></canvas>
          </div>
        </el-card>
      </el-tab-pane>

      <!-- 批量操作 -->
      <el-tab-pane label="批量操作" name="batch">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="executeBatchOperation">
            <el-icon><el-icon-check /></el-icon>
            <span>执行操作</span>
          </el-button>
        </div>

        <!-- 批量操作设置 -->
        <el-card shadow="hover" class="batch-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">批量操作设置</h3>
            </div>
          </template>
          <el-form :model="batchOperationForm" label-width="120px" class="batch-form">
            <el-form-item label="操作类型">
              <el-select v-model="batchOperationForm.operation" placeholder="请选择操作类型">
                <el-option label="重置密码" value="reset_password" />
                <el-option label="启用账户" value="enable_account" />
                <el-option label="禁用账户" value="disable_account" />
                <el-option label="修改会员等级" value="change_level" />
              </el-select>
            </el-form-item>
            
            <el-form-item label="会员等级" v-if="batchOperationForm.operation === 'change_level'">
              <el-select v-model="batchOperationForm.level" placeholder="请选择会员等级">
                <el-option label="普通用户" value="normal" />
                <el-option label="VIP会员" value="vip" />
                <el-option label="SVIP会员" value="svip" />
              </el-select>
            </el-form-item>
            
            <el-form-item label="操作范围">
              <el-radio-group v-model="batchOperationForm.scope">
                <el-radio label="all">所有用户</el-radio>
                <el-radio label="filter">按条件筛选</el-radio>
                <el-radio label="specific">指定用户ID</el-radio>
              </el-radio-group>
            </el-form-item>
            
            <el-form-item label="筛选条件" v-if="batchOperationForm.scope === 'filter'">
              <el-select v-model="batchOperationForm.filterCondition" placeholder="请选择筛选条件">
                <el-option label="注册时间在7天内" value="registered_7days" />
                <el-option label="30天未登录" value="inactive_30days" />
                <el-option label="普通用户" value="level_normal" />
                <el-option label="VIP会员" value="level_vip" />
                <el-option label="SVIP会员" value="level_svip" />
              </el-select>
            </el-form-item>
            
            <el-form-item label="用户ID列表" v-if="batchOperationForm.scope === 'specific'">
              <el-input v-model="batchOperationForm.userIds" placeholder="请输入用户ID，多个用逗号分隔" type="textarea" :rows="3" />
            </el-form-item>
            
            <el-form-item label="操作原因">
              <el-input v-model="batchOperationForm.reason" placeholder="请输入操作原因" />
            </el-form-item>
          </el-form>
        </el-card>

        <!-- 操作预览 -->
        <el-card shadow="hover" class="preview-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">操作预览</h3>
            </div>
          </template>
          <div class="preview-info">
            <div class="preview-item">
              <span class="preview-label">预计影响用户数：</span>
              <span class="preview-value">{{ estimatedAffectedUsers }}</span>
            </div>
            <div class="preview-item">
              <span class="preview-label">操作类型：</span>
              <span class="preview-value">{{ getOperationText(batchOperationForm.operation) }}</span>
            </div>
            <div class="preview-item">
              <span class="preview-label">操作范围：</span>
              <span class="preview-value">{{ getScopeText(batchOperationForm.scope) }}</span>
            </div>
            <div class="preview-item">
              <span class="preview-label">操作原因：</span>
              <span class="preview-value">{{ batchOperationForm.reason || '未指定' }}</span>
            </div>
          </div>
        </el-card>
      </el-tab-pane>
    </el-tabs>

    <!-- 修改用户信息模态框 -->
    <el-dialog
      v-model="editDialogVisible"
      title="修改用户信息"
      width="600px"
      :close-on-click-modal="false"
    >
      <el-form v-model="editForm" label-position="top" class="edit-form">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="用户ID" prop="id">
              <el-input v-model="editForm.id" readonly />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="姓名" prop="name">
              <el-input v-model="editForm.name" placeholder="请输入姓名" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="手机号" prop="fullPhone">
              <el-input v-model="editForm.fullPhone" placeholder="请输入手机号" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="会员状态" prop="memberStatus">
              <el-select v-model="editForm.memberStatus" placeholder="请选择会员状态">
                <el-option label="普通用户" value="normal" />
                <el-option label="VIP会员" value="vip" />
                <el-option label="SVIP会员" value="svip" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="账户状态" prop="accountStatus">
              <el-select v-model="editForm.accountStatus" placeholder="请选择账户状态">
                <el-option label="已启用" value="active" />
                <el-option label="已禁用" value="inactive" />
                <el-option label="已封禁" value="banned" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="健康评分" prop="healthScore">
              <el-input v-model.number="editForm.healthScore" placeholder="请输入健康评分" type="number" min="0" max="100" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="体质类型" prop="constitution">
          <div class="constitution-checkbox-group">
            <el-checkbox-group v-model="editForm.constitution">
              <el-checkbox-button v-for="item in constitutionOptions" :key="item.value" :label="item.label">
                {{ item.label }}
              </el-checkbox-button>
            </el-checkbox-group>
          </div>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="editDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="saveUserChanges">保存修改</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 批量操作对话框 -->
    <el-dialog
      v-model="batchOperationDialogVisible"
      title="批量操作"
      width="600px"
      :close-on-click-modal="false"
    >
      <el-form :model="batchOperationForm" label-width="120px" class="batch-form">
        <el-form-item label="操作类型">
          <el-select v-model="batchOperationForm.operation" placeholder="请选择操作类型">
            <el-option label="重置密码" value="reset_password" />
            <el-option label="启用账户" value="enable_account" />
            <el-option label="禁用账户" value="disable_account" />
            <el-option label="修改会员等级" value="change_level" />
          </el-select>
        </el-form-item>
        
        <el-form-item label="会员等级" v-if="batchOperationForm.operation === 'change_level'">
          <el-select v-model="batchOperationForm.level" placeholder="请选择会员等级">
            <el-option label="普通用户" value="normal" />
            <el-option label="VIP会员" value="vip" />
            <el-option label="SVIP会员" value="svip" />
          </el-select>
        </el-form-item>
        
        <el-form-item label="操作原因">
          <el-input v-model="batchOperationForm.reason" placeholder="请输入操作原因" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="batchOperationDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="confirmBatchOperation">确定执行</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { ElMessage } from 'element-plus'
import * as XLSX from 'xlsx'
import { Chart, registerables } from 'chart.js'
import {
  Document,
  Search,
  RefreshRight,
  View,
  Key,
  Refresh,
  Download,
  Operation
} from '@element-plus/icons-vue'

// 注册Chart.js组件
Chart.register(...registerables)

// 定义组件别名
const ElIconDocument = Document
const ElIconSearch = Search
const ElIconRefreshRight = RefreshRight
const ElIconView = View
const ElIconKey = Key
const ElIconRefresh = Refresh
const ElIconDocumentDownload = Download
const ElIconSOperation = Operation

// 响应式数据
const activeTab = ref('list')
const currentDateRange = ref(30)

// 图表引用
const growthTrendChart = ref(null)
const memberDistributionChart = ref(null)
const activityChart = ref(null)

// 图表实例
let growthTrendChartInstance = null
let memberDistributionChartInstance = null
let activityChartInstance = null

// 体质类型选项
const constitutionOptions = [
  { label: '平和质', value: '平和质' },
  { label: '气虚质', value: '气虚质' },
  { label: '阳虚质', value: '阳虚质' },
  { label: '阴虚质', value: '阴虚质' },
  { label: '痰湿质', value: '痰湿质' },
  { label: '湿热质', value: '湿热质' },
  { label: '血瘀质', value: '血瘀质' },
  { label: '气郁质', value: '气郁质' },
  { label: '特禀质', value: '特禀质' }
]

// 用户数据
const users = ref([
  { id: '100001', name: '李晓明', phone: '138****1234', fullPhone: '13812341234', memberStatus: 'svip', accountStatus: 'active', registerTime: '2024-01-15 09:30', lastLogin: '2024-12-27 14:25', healthScore: 92, constitution: ['平和质', '气虚质', '阳虚质'] },
  { id: '100002', name: '王芳', phone: '139****5678', fullPhone: '13956785678', memberStatus: 'vip', accountStatus: 'active', registerTime: '2024-02-20 14:15', lastLogin: '2024-12-26 18:42', healthScore: 85, constitution: ['痰湿质', '阴虚质'] },
  { id: '100003', name: '张伟', phone: '136****9012', fullPhone: '13690129012', memberStatus: 'normal', accountStatus: 'active', registerTime: '2024-03-10 08:45', lastLogin: '2024-12-27 10:18', healthScore: 78, constitution: ['湿热质', '血瘀质'] },
  { id: '100004', name: '刘洋', phone: '137****3456', fullPhone: '13734563456', memberStatus: 'svip', accountStatus: 'inactive', registerTime: '2024-04-05 16:30', lastLogin: '2024-12-25 22:15', healthScore: 88, constitution: ['平和质', '气郁质'] },
  { id: '100005', name: '陈静', phone: '135****7890', fullPhone: '13578907890', memberStatus: 'vip', accountStatus: 'banned', registerTime: '2024-05-18 11:20', lastLogin: '2024-12-27 08:55', healthScore: 80, constitution: ['痰湿质', '湿热质', '血瘀质'] },
  { id: '100006', name: '赵强', phone: '134****2345', fullPhone: '13423452345', memberStatus: 'normal', accountStatus: 'active', registerTime: '2024-06-25 13:45', lastLogin: '2024-12-27 09:30', healthScore: 75, constitution: ['气虚质', '阳虚质'] },
  { id: '100007', name: '孙丽', phone: '133****6789', fullPhone: '13367896789', memberStatus: 'svip', accountStatus: 'active', registerTime: '2024-07-12 10:20', lastLogin: '2024-12-26 16:45', healthScore: 95, constitution: ['平和质', '阴虚质'] },
  { id: '100008', name: '周杰', phone: '132****0123', fullPhone: '13201230123', memberStatus: 'vip', accountStatus: 'active', registerTime: '2024-08-30 15:10', lastLogin: '2024-12-27 11:20', healthScore: 82, constitution: ['痰湿质', '气郁质'] },
  { id: '100009', name: '吴敏', phone: '131****4567', fullPhone: '13145674567', memberStatus: 'normal', accountStatus: 'active', registerTime: '2024-09-18 09:50', lastLogin: '2024-12-25 14:30', healthScore: 79, constitution: ['湿热质', '阳虚质'] },
  { id: '100010', name: '郑华', phone: '130****8901', fullPhone: '13089018901', memberStatus: 'svip', accountStatus: 'active', registerTime: '2024-10-22 14:25', lastLogin: '2024-12-27 13:15', healthScore: 90, constitution: ['平和质', '血瘀质'] }
])

// 搜索表单
const searchForm = ref({
  id: '',
  phone: '',
  name: '',
  memberStatus: '',
  accountStatus: ''
})

// 分页数据
const currentPage = ref(1)
const pageSize = ref(5)

// 过滤后的用户列表
const filteredUsers = computed(() => {
  return users.value.filter(user => {
    // 多条件AND搜索
    const matchId = !searchForm.value.id || user.id.includes(searchForm.value.id)
    const matchPhone = !searchForm.value.phone || user.fullPhone.includes(searchForm.value.phone)
    const matchName = !searchForm.value.name || user.name.includes(searchForm.value.name)
    const matchMemberStatus = !searchForm.value.memberStatus || user.memberStatus === searchForm.value.memberStatus
    const matchAccountStatus = !searchForm.value.accountStatus || user.accountStatus === searchForm.value.accountStatus
    return matchId && matchPhone && matchName && matchMemberStatus && matchAccountStatus
  })
})

// 当前页用户数据
const currentPageUsers = computed(() => {
  const startIndex = (currentPage.value - 1) * pageSize.value
  const endIndex = startIndex + pageSize.value
  return filteredUsers.value.slice(startIndex, endIndex)
})

// 选中的会员
const selectedMember = ref(null)

// 编辑模态框
const editDialogVisible = ref(false)
const editForm = ref({})

// 批量操作
const batchOperationDialogVisible = ref(false)
const batchOperationForm = ref({
  operation: 'reset_password',
  level: 'normal',
  scope: 'all',
  filterCondition: '',
  userIds: '',
  reason: ''
})

// 会员分布数据
const memberDistributionData = [
  { level: 'normal', name: '普通用户', count: 9330, percentage: 74.2 },
  { level: 'vip', name: 'VIP会员', count: 2250, percentage: 17.9 },
  { level: 'svip', name: 'SVIP会员', count: 1000, percentage: 7.9 }
]

// 预计影响用户数
const estimatedAffectedUsers = computed(() => {
  if (batchOperationForm.value.scope === 'all') {
    return users.value.length
  } else if (batchOperationForm.value.scope === 'filter') {
    return Math.floor(users.value.length * 0.3)
  } else {
    return batchOperationForm.value.userIds.split(',').length
  }
})

// 格式化数字
const formatNumber = (num) => {
  return num.toLocaleString()
}

// 搜索用户
const searchUsers = () => {
  currentPage.value = 1 // 搜索后重置到第一页
  ElMessage.success('搜索完成')
}

// 重置搜索
const resetSearch = () => {
  searchForm.value = {
    id: '',
    phone: '',
    name: '',
    memberStatus: '',
    accountStatus: ''
  }
  currentPage.value = 1
  ElMessage.success('搜索条件已重置')
}

// 查看用户详情
const viewUserDetail = (member) => {
  selectedMember.value = member
  activeTab.value = 'detail'
}

// 打开编辑模态框
const openEditModal = () => {
  if (selectedMember.value) {
    // 深拷贝用户数据到编辑表单
    editForm.value = JSON.parse(JSON.stringify(selectedMember.value))
    editDialogVisible.value = true
  }
}

// 保存用户修改
const saveUserChanges = () => {
  // 更新用户数据
  const index = users.value.findIndex(user => user.id === editForm.value.id)
  if (index !== -1) {
    users.value[index] = { ...editForm.value }
    // 更新当前查看的用户数据
    if (selectedMember.value && selectedMember.value.id === editForm.value.id) {
      selectedMember.value = { ...editForm.value }
    }
    editDialogVisible.value = false
    ElMessage.success('用户信息已更新')
  }
}

// 重置密码
const resetPassword = (userId) => {
  ElMessage.warning('密码重置功能开发中')
}

// 切换用户状态
const toggleStatus = (user) => {
  const newStatus = user.accountStatus === 'active' ? 'inactive' : 'active'
  const index = users.value.findIndex(u => u.id === user.id)
  if (index !== -1) {
    users.value[index].accountStatus = newStatus
    if (selectedMember.value && selectedMember.value.id === user.id) {
      selectedMember.value.accountStatus = newStatus
    }
    ElMessage.success(`用户状态已切换为${newStatus === 'active' ? '已启用' : '已禁用'}`)
  }
}

// 封禁/解封用户
const banUser = (userId) => {
  const user = users.value.find(u => u.id === userId)
  if (user) {
    const newStatus = user.accountStatus === 'banned' ? 'active' : 'banned'
    user.accountStatus = newStatus
    if (selectedMember.value && selectedMember.value.id === userId) {
      selectedMember.value.accountStatus = newStatus
    }
    ElMessage.success(`用户已${newStatus === 'banned' ? '封禁' : '解封'}`)
  }
}

// 分页大小变化
const handleSizeChange = (newSize) => {
  pageSize.value = newSize
  currentPage.value = 1 // 切换每页条数后回到第一页
}

// 当前页码变化
const handleCurrentChange = (newPage) => {
  currentPage.value = newPage
}

// 导出Excel
const exportExcel = () => {
  ElMessage.info('报表导出中...')
  
  // 准备导出数据
  const exportData = users.value.map(user => ({
    '用户ID': user.id,
    '姓名': user.name,
    '手机号': user.fullPhone,
    '注册时间': user.registerTime,
    '最后登录': user.lastLogin,
    '会员状态': getMemberStatusText(user.memberStatus),
    '账户状态': getAccountStatusText(user.accountStatus),
    '健康评分': user.healthScore,
    '体质类型': user.constitution.join(', ')
  }))
  
  // 创建工作簿
  const wb = XLSX.utils.book_new()
  const ws = XLSX.utils.json_to_sheet(exportData)
  
  // 添加工作表
  XLSX.utils.book_append_sheet(wb, ws, '用户数据')
  
  // 导出文件
  setTimeout(() => {
    XLSX.writeFile(wb, `用户列表_${new Date().toISOString().slice(0, 10)}.xlsx`)
    ElMessage.success('报表导出成功！')
  }, 1000)
}

// 打开批量操作对话框
const openBatchOperationDialog = () => {
  batchOperationDialogVisible.value = true
}

// 确认批量操作
const confirmBatchOperation = () => {
  batchOperationDialogVisible.value = false
  ElMessage.success('批量操作已执行')
}

// 执行批量操作
const executeBatchOperation = () => {
  ElMessage.success('批量操作已执行')
}

// 导出分析报表
const exportAnalysisReport = () => {
  ElMessage.info('分析报表导出中...')
  
  // 准备导出数据
  const reportData = {
    '日期范围': `${currentDateRange.value}天`,
    '导出时间': new Date().toLocaleString(),
    '总用户数': '12,580',
    '本月新增': '1,250',
    '活跃用户': '8,920',
    '付费会员': '3,250'
  }
  
  // 创建工作簿
  const wb = XLSX.utils.book_new()
  const overviewSheet = XLSX.utils.json_to_sheet([reportData])
  XLSX.utils.book_append_sheet(wb, overviewSheet, '概览')
  
  // 会员分布数据
  const memberSheetData = memberDistributionData.map(item => ({
    '会员等级': item.name,
    '用户数': item.count,
    '占比': `${item.percentage}%`
  }))
  const memberSheet = XLSX.utils.json_to_sheet(memberSheetData)
  XLSX.utils.book_append_sheet(wb, memberSheet, '会员分布')
  
  // 导出文件
  setTimeout(() => {
    XLSX.writeFile(wb, `用户分析报表_${currentDateRange.value}天_${new Date().toISOString().slice(0, 10)}.xlsx`)
    ElMessage.success('分析报表导出成功！')
  }, 1000)
}

// 获取会员状态文本
const getMemberStatusText = (status) => {
  const map = {
    normal: '普通用户',
    vip: 'VIP会员',
    svip: 'SVIP会员'
  }
  return map[status] || status
}

// 获取会员状态标签类型
const getMemberStatusType = (status) => {
  const map = {
    normal: 'info',
    vip: 'warning',
    svip: 'success'
  }
  return map[status] || 'info'
}

// 获取账户状态文本
const getAccountStatusText = (status) => {
  const map = {
    active: '已启用',
    inactive: '已禁用',
    banned: '已封禁'
  }
  return map[status] || status
}

// 获取账户状态标签类型
const getAccountStatusType = (status) => {
  const map = {
    active: 'success',
    inactive: 'warning',
    banned: 'danger'
  }
  return map[status] || 'info'
}

// 根据用户ID生成头像颜色
const getUserAvatarColor = (id, isSecondary = false) => {
  const colors = ['#667eea', '#764ba2', '#48bb78', '#ed8936', '#f56565']
  const hash = id.split('').reduce((acc, char) => acc + char.charCodeAt(0), 0)
  const index = hash % colors.length
  return isSecondary ? colors[(index + 1) % colors.length] : colors[index]
}

// 获取操作文本
const getOperationText = (operation) => {
  const map = {
    reset_password: '重置密码',
    enable_account: '启用账户',
    disable_account: '禁用账户',
    change_level: '修改会员等级'
  }
  return map[operation] || operation
}

// 获取范围文本
const getScopeText = (scope) => {
  const map = {
    all: '所有用户',
    filter: '按条件筛选',
    specific: '指定用户ID'
  }
  return map[scope] || scope
}

// 初始化用户增长趋势图表
const initGrowthTrendChart = () => {
  if (growthTrendChart.value) {
    if (growthTrendChartInstance) {
      growthTrendChartInstance.destroy()
    }
    const ctx = growthTrendChart.value.getContext('2d')
    
    let labels, newUsers, activeUsers
    if (currentDateRange.value === 7) {
      labels = ['1日', '2日', '3日', '4日', '5日', '6日', '7日']
      newUsers = [120, 145, 132, 158, 172, 185, 168]
      activeUsers = [850, 920, 880, 1050, 1120, 1200, 1080]
    } else if (currentDateRange.value === 30) {
      labels = ['1日', '5日', '10日', '15日', '20日', '25日', '30日']
      newUsers = [120, 580, 1050, 1520, 2080, 2650, 3280]
      activeUsers = [850, 4200, 7800, 11500, 15200, 19800, 24500]
    } else {
      labels = ['1日', '15日', '30日', '45日', '60日', '75日', '90日']
      newUsers = [120, 2850, 5800, 8950, 12200, 15650, 19300]
      activeUsers = [850, 21000, 43500, 67500, 93000, 120000, 148500]
    }
    
    growthTrendChartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [{
          label: '新增用户',
          data: newUsers,
          borderColor: '#667eea',
          backgroundColor: 'rgba(102, 126, 234, 0.1)',
          fill: true,
          tension: 0.4
        }, {
          label: '活跃用户',
          data: activeUsers,
          borderColor: '#48bb78',
          backgroundColor: 'rgba(72, 187, 120, 0.1)',
          fill: true,
          tension: 0.4
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            display: false
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            grid: {
              color: 'rgba(0, 0, 0, 0.05)'
            },
            ticks: {
              callback: function(value) {
                return value.toLocaleString()
              }
            }
          },
          x: {
            grid: {
              display: false
            }
          }
        }
      }
    })
  }
}

// 初始化会员分布图表
const initMemberDistributionChart = () => {
  if (memberDistributionChart.value) {
    if (memberDistributionChartInstance) {
      memberDistributionChartInstance.destroy()
    }
    const ctx = memberDistributionChart.value.getContext('2d')
    
    const labels = memberDistributionData.map(item => item.name)
    const data = memberDistributionData.map(item => item.count)
    const colors = ['#667eea', '#48bb78', '#ed8936']
    
    memberDistributionChartInstance = new Chart(ctx, {
      type: 'pie',
      data: {
        labels: labels,
        datasets: [{
          data: data,
          backgroundColor: colors,
          borderWidth: 0
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            display: false
          }
        }
      }
    })
  }
}

// 初始化用户活跃度图表
const initActivityChart = () => {
  if (activityChart.value) {
    if (activityChartInstance) {
      activityChartInstance.destroy()
    }
    const ctx = activityChart.value.getContext('2d')
    
    let labels, loginData, operationData
    if (currentDateRange.value === 7) {
      labels = ['1日', '2日', '3日', '4日', '5日', '6日', '7日']
      loginData = [1200, 1350, 1420, 1380, 1520, 1650, 1720]
      operationData = [3500, 3800, 4200, 4000, 4500, 4800, 5200]
    } else if (currentDateRange.value === 30) {
      labels = ['1日', '5日', '10日', '15日', '20日', '25日', '30日']
      loginData = [1200, 5800, 10500, 15200, 20800, 26500, 32800]
      operationData = [3500, 18000, 35000, 52000, 71000, 92000, 115000]
    } else {
      labels = ['1日', '15日', '30日', '45日', '60日', '75日', '90日']
      loginData = [1200, 28500, 58000, 89500, 122000, 156500, 193000]
      operationData = [3500, 168000, 345000, 532000, 739000, 966000, 1213000]
    }
    
    activityChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: labels,
        datasets: [{
          label: '登录次数',
          data: loginData,
          backgroundColor: '#667eea',
          borderRadius: 8,
          borderSkipped: false
        }, {
          label: '操作次数',
          data: operationData,
          backgroundColor: '#48bb78',
          borderRadius: 8,
          borderSkipped: false
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            display: false
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            grid: {
              color: 'rgba(0, 0, 0, 0.05)'
            },
            ticks: {
              callback: function(value) {
                return value.toLocaleString()
              }
            }
          },
          x: {
            grid: {
              display: false
            }
          }
        }
      }
    })
  }
}

// 更新所有图表
const updateAllCharts = () => {
  initGrowthTrendChart()
  initMemberDistributionChart()
  initActivityChart()
  ElMessage.success('图表数据已更新')
}

// 生命周期钩子
onMounted(() => {
  initGrowthTrendChart()
  initMemberDistributionChart()
  initActivityChart()
})

onUnmounted(() => {
  // 清理图表实例
  if (growthTrendChartInstance) {
    growthTrendChartInstance.destroy()
  }
  if (memberDistributionChartInstance) {
    memberDistributionChartInstance.destroy()
  }
  if (activityChartInstance) {
    activityChartInstance.destroy()
  }
})
</script>

<style scoped>
.users-container {
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

.search-card {
  margin-bottom: 20px;
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
}

.section-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--el-text-color-primary);
  margin: 0 0 20px 0;
}

.search-form {
  width: 100%;
}

.form-row {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-bottom: 20px;
}

.form-row :deep(.el-form-item) {
  margin-right: 20px;
}

.form-actions {
  display: flex;
  gap: 16px;
  justify-content: flex-end;
}

.users-table-card {
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
}

.table-container {
  margin-bottom: 20px;
  overflow-x: auto;
}

:deep(.el-table__header-wrapper) {
  background-color: var(--el-bg-color-page);
}

.pagination-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 20px;
  border-top: 1px solid var(--el-border-color);
}

.pagination-info {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

/* 用户信息 */
.user-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.user-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: 600;
  font-size: 14px;
}

.user-avatar.large {
  width: 80px;
  height: 80px;
  font-size: 28px;
}

.user-details {
  display: flex;
  flex-direction: column;
}

.user-name {
  font-size: 14px;
  font-weight: 500;
  color: var(--el-text-color-primary);
  margin-bottom: 4px;
}

.user-id {
  font-size: 12px;
  color: var(--el-text-color-tertiary);
}

/* 详情页 */
.detail-card {
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
}

.card-header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--el-text-color-primary);
  margin: 0;
}

.user-detail {
  padding: 10px 0;
}

.detail-header {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 1px solid var(--el-border-color);
}

.user-info-detail {
  display: flex;
  flex-direction: column;
}

.user-info-detail .user-name {
  font-size: 20px;
  margin-bottom: 8px;
}

.user-basic-info {
  display: flex;
  gap: 20px;
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.member-status-info {
  margin-top: 8px;
}

.detail-sections {
  margin-top: 20px;
}

.detail-section {
  margin-bottom: 30px;
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
}

.detail-item {
  display: flex;
  align-items: center;
  gap: 12px;
}

.detail-label {
  font-size: 14px;
  color: var(--el-text-color-secondary);
  min-width: 100px;
}

.detail-value {
  font-size: 14px;
  color: var(--el-text-color-primary);
  font-weight: 500;
}

.card-actions {
  display: flex;
  gap: 16px;
  justify-content: flex-end;
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid var(--el-border-color);
}

.empty-detail {
  padding: 60px 0;
  text-align: center;
}

/* 分析页面 */
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
}

.stat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.stat-icon {
  font-size: 24px;
}

.stat-trend {
  font-size: 12px;
  font-weight: 600;
}

.stat-trend.up {
  color: var(--el-color-success);
}

.stat-trend.down {
  color: var(--el-color-danger);
}

.stat-value {
  font-size: 24px;
  font-weight: 700;
  color: var(--el-text-color-primary);
  margin-bottom: 4px;
}

.stat-label {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.grid-2 {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.chart-card {
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
  margin-bottom: 20px;
}

.chart-desc {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.chart-container {
  height: 300px;
  padding: 10px;
}

.member-distribution-info {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  padding: 20px 0;
}

.member-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 16px;
  background-color: var(--el-bg-color-page);
  border-radius: var(--el-border-radius-base);
}

.member-name {
  font-size: 14px;
  font-weight: 500;
  color: var(--el-text-color-primary);
}

.member-stats {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.member-stats .stat {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.member-stats .stat-label {
  font-size: 12px;
  color: var(--el-text-color-tertiary);
}

.member-stats .stat-value {
  font-size: 14px;
  font-weight: 600;
  color: var(--el-color-primary);
}

/* 批量操作 */
.batch-card {
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
  margin-bottom: 20px;
}

.batch-form {
  padding: 20px 0;
}

.preview-card {
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
}

.preview-info {
  padding: 20px 0;
}

.preview-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid var(--el-border-color);
}

.preview-item:last-child {
  border-bottom: none;
}

.preview-label {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.preview-value {
  font-size: 14px;
  font-weight: 500;
  color: var(--el-text-color-primary);
}

/* 编辑表单 */
.edit-form {
  width: 100%;
}

.constitution-checkbox-group {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .grid-2 {
    grid-template-columns: 1fr;
  }
  
  .form-row {
    flex-direction: column;
  }
  
  .form-row :deep(.el-form-item) {
    margin-right: 0;
    width: 100%;
  }
  
  .detail-grid {
    grid-template-columns: 1fr;
  }
  
  .member-distribution-info {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 768px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }
  
  .pagination-container {
    flex-direction: column;
    align-items: flex-start;
    gap: 16px;
  }
  
  .top-actions {
    flex-direction: column;
    align-items: stretch;
  }
  
  .detail-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 20px;
  }
  
  .member-distribution-info {
    grid-template-columns: 1fr;
  }
  
  .card-actions {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>