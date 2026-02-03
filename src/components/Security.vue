<!--
安全管理组件
API配置信息：
- 开发环境：http://localhost:8080/api/v1
- 生产环境：https://api.lingjing.com/v1
- 请求超时时间：30秒
对应API接口：
- GET /api/security/score - 获取安全评分
- GET /api/security/events - 获取安全事件列表
- GET /api/security/events/:id - 获取安全事件详情
- PUT /api/security/events/:id/status - 更新事件状态
- GET /api/security/configs - 获取安全配置
- PUT /api/security/configs - 更新安全配置
- GET /api/security/statistics - 获取安全统计数据
- GET /api/security/report - 获取安全报表
-->
<template>
  <div class="security-container">
    <!-- 页面标题和操作 -->
    <div class="header-section">
      <h1 class="page-title">安全管理</h1>
      <p class="page-subtitle">系统安全评分、事件监控与配置管理</p>
    </div>

    <!-- 标签页切换 -->
    <el-tabs v-model="activeTab" class="main-tabs">
      <!-- 安全概览 -->
      <el-tab-pane label="安全概览" name="overview">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <div class="date-range">
            <el-select v-model="currentDateRange" placeholder="选择日期范围" size="large" @change="updateAllCharts">
              <el-option label="最近7天" :value="7" />
              <el-option label="最近30天" :value="30" />
              <el-option label="最近90天" :value="90" />
            </el-select>
          </div>
          <el-button type="primary" size="large" @click="exportSecurityReport">
            <el-icon><el-icon-document-download /></el-icon>
            <span>导出报表</span>
          </el-button>
        </div>

        <!-- 安全评分与监控看板 -->
        <div class="security-score-section">
          <el-card shadow="hover" class="score-card">
            <div class="score-header">
              <div>
                <h3 class="score-title">安全评分</h3>
                <p class="score-subtitle">系统整体安全状况评估</p>
              </div>
              <div class="score-circle">
                <span class="score-value">92</span>
                <span class="score-unit">/100</span>
              </div>
            </div>
            <div class="score-metrics">
              <div class="metric-item">
                <div class="metric-value">{{ formatNumber(12345) }}</div>
                <div class="metric-label">今日请求数</div>
              </div>
              <div class="metric-item">
                <div class="metric-value">99.9%</div>
                <div class="metric-label">系统可用性</div>
              </div>
              <div class="metric-item">
                <div class="metric-value">{{ formatNumber(156) }}</div>
                <div class="metric-label">今日拦截</div>
              </div>
              <div class="metric-item">
                <div class="metric-value">{{ formatNumber(3) }}</div>
                <div class="metric-label">待处理告警</div>
              </div>
            </div>
          </el-card>
        </div>

        <!-- 安全事件趋势与分布 -->
        <div class="grid-2">
          <!-- 安全事件趋势 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">安全事件趋势</h3>
                <span class="chart-desc">过去 {{ currentDateRange }} 天的安全事件变化</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="securityTrendChart"></canvas>
            </div>
          </el-card>
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">安全事件分布</h3>
                <span class="chart-desc">不同类型安全事件占比</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="securityDistributionChart"></canvas>
            </div>
            <div class="event-types">
              <div class="event-item" v-for="item in securityEventTypes" :key="item.type">
                <div class="event-name">{{ item.name }}</div>
                <div class="event-count">{{ item.count }} 次</div>
              </div>
            </div>
          </el-card>
        </div>

        <!-- 安全配置与合规检查 -->
        <div class="security-config-section">
          <el-card shadow="hover" class="config-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">数据安全配置</h3>
                <span class="card-action">查看详情 →</span>
              </div>
            </template>
            <div class="config-grid">
              <div class="config-item">
                <div class="config-icon success">🔒</div>
                <div class="config-info">
                  <div class="config-name">敏感数据加密</div>
                  <div class="config-status">已启用</div>
                </div>
              </div>
              <div class="config-item">
                <div class="config-icon success">🌐</div>
                <div class="config-info">
                  <div class="config-name">HTTPS/TLS</div>
                  <div class="config-status">已启用</div>
                </div>
              </div>
              <div class="config-item">
                <div class="config-icon success">🗝️</div>
                <div class="config-info">
                  <div class="config-name">密钥管理</div>
                  <div class="config-status">已启用</div>
                </div>
              </div>
              <div class="config-item">
                <div class="config-icon warning">🔍</div>
                <div class="config-info">
                  <div class="config-name">异常登录检测</div>
                  <div class="config-status">待优化</div>
                </div>
              </div>
            </div>
          </el-card>

          <el-card shadow="hover" class="config-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户隐私与合规</h3>
                <span class="card-action">查看详情 →</span>
              </div>
            </template>
            <div class="config-grid">
              <div class="config-item">
                <div class="config-icon success">📜</div>
                <div class="config-info">
                  <div class="config-name">隐私政策</div>
                  <div class="config-status">已更新</div>
                </div>
              </div>
              <div class="config-item">
                <div class="config-icon success">🔑</div>
                <div class="config-info">
                  <div class="config-name">权限最小化</div>
                  <div class="config-status">已启用</div>
                </div>
              </div>
              <div class="config-item">
                <div class="config-icon success">👤</div>
                <div class="config-info">
                  <div class="config-name">数据匿名化</div>
                  <div class="config-status">已启用</div>
                </div>
              </div>
              <div class="config-item">
                <div class="config-icon success">🧪</div>
                <div class="config-info">
                  <div class="config-name">第三方审计</div>
                  <div class="config-status">已完成</div>
                </div>
              </div>
            </div>
          </el-card>
        </div>
      </el-tab-pane>

      <!-- 安全告警 -->
      <el-tab-pane label="安全告警" name="alerts">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="openAlertSettingsDialog">
            <el-icon><el-icon-setting /></el-icon>
            <span>告警设置</span>
          </el-button>
          <el-button @click="exportAlerts">
            <el-icon><el-icon-document-download /></el-icon>
            <span>导出告警</span>
          </el-button>
        </div>

        <!-- 搜索卡片 -->
        <el-card shadow="hover" class="search-card">
          <h3 class="section-title">搜索条件</h3>
          <div class="search-form">
            <div class="form-row">
              <el-form-item label="告警级别">
                <el-select v-model="alertSearchForm.level" placeholder="请选择告警级别" clearable>
                  <el-option label="危险" value="danger" />
                  <el-option label="警告" value="warning" />
                  <el-option label="信息" value="info" />
                </el-select>
              </el-form-item>
              <el-form-item label="告警类型">
                <el-select v-model="alertSearchForm.type" placeholder="请选择告警类型" clearable>
                  <el-option label="暴力破解" value="brute_force" />
                  <el-option label="SQL注入" value="sql_injection" />
                  <el-option label="跨站脚本" value="xss" />
                  <el-option label="DDoS攻击" value="ddos" />
                  <el-option label="异常访问" value="anomaly" />
                </el-select>
              </el-form-item>
              <el-form-item label="处理状态">
                <el-select v-model="alertSearchForm.status" placeholder="请选择处理状态" clearable>
                  <el-option label="未处理" value="pending" />
                  <el-option label="已处理" value="processed" />
                  <el-option label="已忽略" value="ignored" />
                </el-select>
              </el-form-item>
              <el-form-item label="时间范围">
                <el-date-picker
                  v-model="alertSearchForm.dateRange"
                  type="daterange"
                  range-separator="至"
                  start-placeholder="开始日期"
                  end-placeholder="结束日期"
                  format="YYYY-MM-DD"
                  value-format="YYYY-MM-DD"
                />
              </el-form-item>
            </div>
            <div class="form-actions">
              <el-button type="primary" @click="searchAlerts">
                <el-icon><el-icon-search /></el-icon>
                <span>搜索</span>
              </el-button>
              <el-button @click="resetAlertSearch">
                <el-icon><el-icon-refresh-right /></el-icon>
                <span>重置</span>
              </el-button>
            </div>
          </div>
        </el-card>

        <!-- 告警列表 -->
        <el-card shadow="hover" class="alerts-card">
          <h3 class="section-title">安全告警列表</h3>
          <div class="table-container">
            <el-table :data="filteredAlerts" stripe style="width: 100%">
              <el-table-column prop="id" label="告警ID" width="100" />
              <el-table-column label="告警级别" width="100">
                <template #default="scope">
                  <el-tag :type="scope.row.level">
                    {{ getAlertLevelText(scope.row.level) }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column label="告警类型" width="120">
                <template #default="scope">
                  <span>{{ getAlertTypeText(scope.row.type) }}</span>
                </template>
              </el-table-column>
              <el-table-column label="告警内容" />
              <el-table-column label="来源IP" width="150" />
              <el-table-column label="发生时间" width="180" />
              <el-table-column label="处理状态" width="120">
                <template #default="scope">
                  <el-tag :type="getAlertStatusType(scope.row.status)">
                    {{ getAlertStatusText(scope.row.status) }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column label="操作" width="150" fixed="right">
                <template #default="scope">
                  <el-button type="primary" link @click="processAlert(scope.row)">
                    <el-icon><el-icon-check /></el-icon>
                    <span>处理</span>
                  </el-button>
                  <el-button type="danger" link @click="ignoreAlert(scope.row.id)">
                    <el-icon><el-icon-delete /></el-icon>
                    <span>忽略</span>
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </div>

          <!-- 分页 -->
          <div class="pagination-container">
            <el-pagination
              v-model:current-page="alertCurrentPage"
              v-model:page-size="alertPageSize"
              :page-sizes="[10, 20, 50, 100]"
              layout="total, sizes, prev, pager, next, jumper"
              :total="filteredAlerts.length"
              @size-change="handleAlertSizeChange"
              @current-change="handleAlertCurrentChange"
            />
          </div>
        </el-card>
      </el-tab-pane>

      <!-- 安全配置 -->
      <el-tab-pane label="安全配置" name="config">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="saveAllConfigs">
            <el-icon><el-icon-check /></el-icon>
            <span>保存所有配置</span>
          </el-button>
        </div>

        <!-- 数据安全配置 -->
        <el-card shadow="hover" class="config-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">数据安全配置</h3>
              <el-button type="primary" size="small" @click="saveDataSecurityConfig">保存配置</el-button>
            </div>
          </template>
          <el-form :model="dataSecurityConfig" label-width="120px" class="config-form">
            <el-form-item label="敏感数据加密">
              <el-switch v-model="dataSecurityConfig.encryption" :active-text="'启用'" :inactive-text="'禁用'" />
            </el-form-item>
            <el-form-item label="API访问控制">
              <el-switch v-model="dataSecurityConfig.apiControl" :active-text="'启用'" :inactive-text="'禁用'" />
            </el-form-item>
            <el-form-item label="IP白名单">
              <el-input v-model="dataSecurityConfig.ipWhitelist" placeholder="多个IP用逗号分隔" type="textarea" :rows="3" />
            </el-form-item>
            <el-form-item label="请求频率限制">
              <el-input-number v-model="dataSecurityConfig.rateLimit" min="0" max="1000" />
              <span class="unit">次/分钟</span>
            </el-form-item>
          </el-form>
        </el-card>

        <!-- 认证安全配置 -->
        <el-card shadow="hover" class="config-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">认证安全配置</h3>
              <el-button type="primary" size="small" @click="saveAuthSecurityConfig">保存配置</el-button>
            </div>
          </template>
          <el-form :model="authSecurityConfig" label-width="120px" class="config-form">
            <el-form-item label="密码复杂度">
              <el-select v-model="authSecurityConfig.passwordComplexity" placeholder="请选择密码复杂度">
                <el-option label="低" value="low" />
                <el-option label="中" value="medium" />
                <el-option label="高" value="high" />
              </el-select>
            </el-form-item>
            <el-form-item label="密码过期时间">
              <el-input-number v-model="authSecurityConfig.passwordExpiry" min="0" max="365" />
              <span class="unit">天</span>
            </el-form-item>
            <el-form-item label="登录失败限制">
              <el-input-number v-model="authSecurityConfig.loginFailLimit" min="0" max="10" />
              <span class="unit">次</span>
            </el-form-item>
            <el-form-item label="登录锁定时间">
              <el-input-number v-model="authSecurityConfig.lockoutDuration" min="0" max="1440" />
              <span class="unit">分钟</span>
            </el-form-item>
            <el-form-item label="双因素认证">
              <el-switch v-model="authSecurityConfig.twoFactorAuth" :active-text="'启用'" :inactive-text="'禁用'" />
            </el-form-item>
          </el-form>
        </el-card>

        <!-- 日志安全配置 -->
        <el-card shadow="hover" class="config-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">日志安全配置</h3>
              <el-button type="primary" size="small" @click="saveLogSecurityConfig">保存配置</el-button>
            </div>
          </template>
          <el-form :model="logSecurityConfig" label-width="120px" class="config-form">
            <el-form-item label="日志级别">
              <el-select v-model="logSecurityConfig.logLevel" placeholder="请选择日志级别">
                <el-option label="调试" value="debug" />
                <el-option label="信息" value="info" />
                <el-option label="警告" value="warn" />
                <el-option label="错误" value="error" />
              </el-select>
            </el-form-item>
            <el-form-item label="日志保留时间">
              <el-input-number v-model="logSecurityConfig.logRetention" min="1" max="365" />
              <span class="unit">天</span>
            </el-form-item>
            <el-form-item label="敏感信息脱敏">
              <el-switch v-model="logSecurityConfig.dataMasking" :active-text="'启用'" :inactive-text="'禁用'" />
            </el-form-item>
            <el-form-item label="日志加密">
              <el-switch v-model="logSecurityConfig.logEncryption" :active-text="'启用'" :inactive-text="'禁用'" />
            </el-form-item>
          </el-form>
        </el-card>
      </el-tab-pane>

      <!-- 安全日志 -->
      <el-tab-pane label="安全日志" name="logs">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="exportLogs">
            <el-icon><el-icon-document-download /></el-icon>
            <span>导出日志</span>
          </el-button>
          <el-button @click="clearLogs">
            <el-icon><el-icon-delete /></el-icon>
            <span>清空日志</span>
          </el-button>
        </div>

        <!-- 搜索卡片 -->
        <el-card shadow="hover" class="search-card">
          <h3 class="section-title">搜索条件</h3>
          <div class="search-form">
            <div class="form-row">
              <el-form-item label="日志类型">
                <el-select v-model="logSearchForm.type" placeholder="请选择日志类型" clearable>
                  <el-option label="登录日志" value="login" />
                  <el-option label="操作日志" value="operation" />
                  <el-option label="安全日志" value="security" />
                  <el-option label="系统日志" value="system" />
                </el-select>
              </el-form-item>
              <el-form-item label="操作人">
                <el-input v-model="logSearchForm.user" placeholder="请输入操作人" clearable />
              </el-form-item>
              <el-form-item label="IP地址">
                <el-input v-model="logSearchForm.ip" placeholder="请输入IP地址" clearable />
              </el-form-item>
              <el-form-item label="时间范围">
                <el-date-picker
                  v-model="logSearchForm.dateRange"
                  type="daterange"
                  range-separator="至"
                  start-placeholder="开始日期"
                  end-placeholder="结束日期"
                  format="YYYY-MM-DD"
                  value-format="YYYY-MM-DD"
                />
              </el-form-item>
            </div>
            <div class="form-actions">
              <el-button type="primary" @click="searchLogs">
                <el-icon><el-icon-search /></el-icon>
                <span>搜索</span>
              </el-button>
              <el-button @click="resetLogSearch">
                <el-icon><el-icon-refresh-right /></el-icon>
                <span>重置</span>
              </el-button>
            </div>
          </div>
        </el-card>

        <!-- 日志列表 -->
        <el-card shadow="hover" class="logs-card">
          <h3 class="section-title">安全日志列表</h3>
          <div class="table-container">
            <el-table :data="filteredLogs" stripe style="width: 100%">
              <el-table-column prop="id" label="日志ID" width="100" />
              <el-table-column label="日志类型" width="120">
                <template #default="scope">
                  <el-tag :type="getLogTypeTag(scope.row.type)">
                    {{ getLogTypeText(scope.row.type) }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="content" label="日志内容" />
              <el-table-column prop="user" label="操作人" width="120" />
              <el-table-column prop="ip" label="IP地址" width="150" />
              <el-table-column prop="userAgent" label="用户代理" width="200" />
              <el-table-column prop="createTime" label="创建时间" width="180" />
            </el-table>
          </div>

          <!-- 分页 -->
          <div class="pagination-container">
            <el-pagination
              v-model:current-page="logCurrentPage"
              v-model:page-size="logPageSize"
              :page-sizes="[10, 20, 50, 100]"
              layout="total, sizes, prev, pager, next, jumper"
              :total="filteredLogs.length"
              @size-change="handleLogSizeChange"
              @current-change="handleLogCurrentChange"
            />
          </div>
        </el-card>
      </el-tab-pane>
    </el-tabs>

    <!-- 告警设置对话框 -->
    <el-dialog
      v-model="alertSettingsDialogVisible"
      title="告警设置"
      width="600px"
    >
      <el-form :model="alertSettings" label-width="120px">
        <el-form-item label="危险告警通知">
          <el-switch v-model="alertSettings.dangerEnabled" :active-text="'启用'" :inactive-text="'禁用'" />
        </el-form-item>
        <el-form-item label="警告告警通知">
          <el-switch v-model="alertSettings.warningEnabled" :active-text="'启用'" :inactive-text="'禁用'" />
        </el-form-item>
        <el-form-item label="信息告警通知">
          <el-switch v-model="alertSettings.infoEnabled" :active-text="'启用'" :inactive-text="'禁用'" />
        </el-form-item>
        <el-form-item label="通知方式">
          <el-checkbox-group v-model="alertSettings.notificationMethods">
            <el-checkbox label="email">邮件</el-checkbox>
            <el-checkbox label="sms">短信</el-checkbox>
            <el-checkbox label="system">系统消息</el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-form-item label="告警阈值">
          <el-input-number v-model="alertSettings.threshold" min="1" max="100" />
          <span class="unit">次/分钟</span>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="alertSettingsDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="saveAlertSettings">保存设置</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { Chart, registerables } from 'chart.js'
import * as XLSX from 'xlsx'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  Download,
  Setting,
  Check,
  Delete,
  Search,
  RefreshRight
} from '@element-plus/icons-vue'

// 注册Chart.js组件
Chart.register(...registerables)

// 定义组件别名
const ElIconDocumentDownload = Download
const ElIconSetting = Setting
const ElIconCheck = Check
const ElIconDelete = Delete
const ElIconSearch = Search
const ElIconRefreshRight = RefreshRight

// 响应式数据
const activeTab = ref('overview')
const currentDateRange = ref(30)

// 图表引用
const securityTrendChart = ref(null)
const securityDistributionChart = ref(null)

// 图表实例
let securityTrendChartInstance = null
let securityDistributionChartInstance = null

// 安全事件类型
const securityEventTypes = [
  { type: 'brute_force', name: '暴力破解', count: 85, color: '#f56565' },
  { type: 'sql_injection', name: 'SQL注入', count: 32, color: '#ed8936' },
  { type: 'xss', name: '跨站脚本', count: 28, color: '#ed8936' },
  { type: 'ddos', name: 'DDoS攻击', count: 15, color: '#f56565' },
  { type: 'malware', name: '恶意软件', count: 8, color: '#f56565' },
  { type: 'anomaly', name: '异常访问', count: 120, color: '#48bb78' }
]

// 安全告警数据
const securityAlerts = [
  { id: 1, level: 'danger', type: 'brute_force', content: 'IP地址 192.168.1.100 连续10次登录失败', ip: '192.168.1.100', time: '2024-12-27 14:32:15', status: 'pending' },
  { id: 2, level: 'warning', type: 'anomaly', content: '用户 ID: 100001 尝试访问未授权数据', ip: '192.168.1.101', time: '2024-12-27 13:45:22', status: 'processed' },
  { id: 3, level: 'info', type: 'xss', content: '检测到潜在的XSS攻击', ip: '192.168.1.102', time: '2024-12-27 12:15:36', status: 'ignored' },
  { id: 4, level: 'danger', type: 'ddos', content: '系统流量异常增长，疑似DDoS攻击', ip: '192.168.1.103', time: '2024-12-27 11:05:48', status: 'pending' },
  { id: 5, level: 'warning', type: 'sql_injection', content: '检测到SQL注入尝试', ip: '192.168.1.104', time: '2024-12-27 10:30:12', status: 'processed' }
]

// 安全日志数据
const securityLogs = [
  { id: 1, type: 'login', content: '用户登录成功', user: 'admin', ip: '192.168.1.1', userAgent: 'Mozilla/5.0', createTime: '2024-12-27 14:30:00' },
  { id: 2, type: 'operation', content: '修改用户信息', user: 'admin', ip: '192.168.1.1', userAgent: 'Mozilla/5.0', createTime: '2024-12-27 14:25:00' },
  { id: 3, type: 'security', content: '检测到暴力破解尝试', user: 'unknown', ip: '192.168.1.100', userAgent: 'Unknown', createTime: '2024-12-27 14:20:00' },
  { id: 4, type: 'system', content: '系统重启', user: 'system', ip: '127.0.0.1', userAgent: 'System', createTime: '2024-12-27 14:15:00' },
  { id: 5, type: 'login', content: '用户登录失败', user: 'user1', ip: '192.168.1.2', userAgent: 'Mozilla/5.0', createTime: '2024-12-27 14:10:00' }
]

// 搜索表单
const alertSearchForm = ref({
  level: '',
  type: '',
  status: ''
})

const logSearchForm = ref({
  type: '',
  user: '',
  ip: '',
  dateRange: []
})

// 分页数据
const alertCurrentPage = ref(1)
const alertPageSize = ref(10)
const logCurrentPage = ref(1)
const logPageSize = ref(10)

// 过滤后的告警
const filteredAlerts = ref(securityAlerts)

// 过滤后的日志
const filteredLogs = ref(securityLogs)

// 告警设置
const alertSettingsDialogVisible = ref(false)
const alertSettings = ref({
  dangerEnabled: true,
  warningEnabled: true,
  infoEnabled: false,
  notificationMethods: ['email', 'system'],
  threshold: 5
})

// 配置数据
const dataSecurityConfig = ref({
  encryption: true,
  apiControl: true,
  ipWhitelist: '192.168.1.0/24, 10.0.0.0/8',
  rateLimit: 100
})

const authSecurityConfig = ref({
  passwordComplexity: 'medium',
  passwordExpiry: 90,
  loginFailLimit: 5,
  lockoutDuration: 30,
  twoFactorAuth: false
})

const logSecurityConfig = ref({
  logLevel: 'info',
  logRetention: 90,
  dataMasking: true,
  logEncryption: false
})

// 格式化数字
const formatNumber = (num) => {
  return num.toLocaleString()
}

// 初始化安全事件趋势图表
const initSecurityTrendChart = () => {
  if (securityTrendChart.value) {
    if (securityTrendChartInstance) {
      securityTrendChartInstance.destroy()
    }
    const ctx = securityTrendChart.value.getContext('2d')
    
    let labels, attackData, failedLoginData
    if (currentDateRange.value === 7) {
      labels = ['1日', '2日', '3日', '4日', '5日', '6日', '7日']
      attackData = [120, 145, 132, 158, 172, 185, 168]
      failedLoginData = [85, 92, 88, 105, 112, 120, 108]
    } else if (currentDateRange.value === 30) {
      labels = ['1日', '5日', '10日', '15日', '20日', '25日', '30日']
      attackData = [120, 580, 1050, 1520, 2080, 2650, 3280]
      failedLoginData = [85, 420, 780, 1150, 1520, 1980, 2450]
    } else {
      labels = ['1日', '15日', '30日', '45日', '60日', '75日', '90日']
      attackData = [120, 2850, 5800, 8950, 12200, 15650, 19300]
      failedLoginData = [85, 2100, 4350, 6750, 9300, 12000, 14850]
    }
    
    securityTrendChartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [{
          label: '总攻击次数',
          data: attackData,
          borderColor: '#f56565',
          backgroundColor: 'rgba(245, 101, 101, 0.1)',
          fill: true,
          tension: 0.4
        }, {
          label: '失败登录次数',
          data: failedLoginData,
          borderColor: '#ed8936',
          backgroundColor: 'rgba(237, 137, 54, 0.1)',
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

// 初始化安全事件分布图表
const initSecurityDistributionChart = () => {
  if (securityDistributionChart.value) {
    if (securityDistributionChartInstance) {
      securityDistributionChartInstance.destroy()
    }
    const ctx = securityDistributionChart.value.getContext('2d')
    
    const labels = securityEventTypes.map(item => item.name)
    const data = securityEventTypes.map(item => item.count)
    const colors = securityEventTypes.map(item => item.color)
    
    securityDistributionChartInstance = new Chart(ctx, {
      type: 'doughnut',
      data: {
        labels: labels,
        datasets: [{
          data: data,
          backgroundColor: colors,
          borderWidth: 0,
          hoverOffset: 4
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
        cutout: '70%'
      }
    })
  }
}

// 更新所有图表
const updateAllCharts = () => {
  initSecurityTrendChart()
  initSecurityDistributionChart()
  ElMessage.success('图表数据已更新')
}

// 导出安全报告
const exportSecurityReport = () => {
  ElMessage.info('安全报告导出中...')
  
  // 准备导出数据
  const reportData = {
    '日期范围': `${currentDateRange.value}天`,
    '导出时间': new Date().toLocaleString(),
    '安全评分': '92/100',
    '今日请求数': '12,345',
    '系统可用性': '99.9%',
    '今日拦截': '156',
    '待处理告警': '3'
  }
  
  // 安全事件数据
  const eventSheetData = securityEventTypes.map(item => ({
    '事件类型': item.name,
    '发生次数': item.count
  }))
  
  // 创建工作簿
  const wb = XLSX.utils.book_new()
  
  // 添加概览工作表
  const overviewSheet = XLSX.utils.json_to_sheet([reportData])
  XLSX.utils.book_append_sheet(wb, overviewSheet, '概览')
  
  // 添加安全事件工作表
  const eventSheet = XLSX.utils.json_to_sheet(eventSheetData)
  XLSX.utils.book_append_sheet(wb, eventSheet, '安全事件')
  
  // 导出文件
  setTimeout(() => {
    XLSX.writeFile(wb, `安全报告_${currentDateRange.value}天_${new Date().toISOString().slice(0, 10)}.xlsx`)
    ElMessage.success('安全报告导出成功！')
  }, 1000)
}

// 搜索告警
const searchAlerts = () => {
  filteredAlerts.value = securityAlerts.filter(alert => {
    const matchLevel = !alertSearchForm.value.level || alert.level === alertSearchForm.value.level
    const matchType = !alertSearchForm.value.type || alert.type === alertSearchForm.value.type
    const matchStatus = !alertSearchForm.value.status || alert.status === alertSearchForm.value.status
    return matchLevel && matchType && matchStatus
  })
  alertCurrentPage.value = 1
  ElMessage.success('搜索完成')
}

// 重置告警搜索
const resetAlertSearch = () => {
  alertSearchForm.value = {
    level: '',
    type: '',
    status: ''
  }
  filteredAlerts.value = securityAlerts
  alertCurrentPage.value = 1
  ElMessage.success('搜索条件已重置')
}

// 搜索日志
const searchLogs = () => {
  filteredLogs.value = securityLogs.filter(log => {
    const matchType = !logSearchForm.value.type || log.type === logSearchForm.value.type
    const matchUser = !logSearchForm.value.user || log.user.includes(logSearchForm.value.user)
    const matchIp = !logSearchForm.value.ip || log.ip.includes(logSearchForm.value.ip)
    return matchType && matchUser && matchIp
  })
  logCurrentPage.value = 1
  ElMessage.success('搜索完成')
}

// 重置日志搜索
const resetLogSearch = () => {
  logSearchForm.value = {
    type: '',
    user: '',
    ip: '',
    dateRange: []
  }
  filteredLogs.value = securityLogs
  logCurrentPage.value = 1
  ElMessage.success('搜索条件已重置')
}

// 处理告警
const processAlert = (alert) => {
  alert.status = 'processed'
  ElMessage.success('告警已处理')
}

// 忽略告警
const ignoreAlert = (alertId) => {
  const alert = securityAlerts.find(a => a.id === alertId)
  if (alert) {
    alert.status = 'ignored'
    ElMessage.success('告警已忽略')
  }
}

// 导出告警
const exportAlerts = () => {
  ElMessage.success('告警导出成功')
}

// 导出日志
const exportLogs = () => {
  ElMessage.success('日志导出成功')
}

// 清空日志
const clearLogs = () => {
  ElMessageBox.confirm('确定要清空所有日志吗？', '警告', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    filteredLogs.value = []
    ElMessage.success('日志已清空')
  }).catch(() => {
    // 取消操作
  })
}

// 保存告警设置
const saveAlertSettings = () => {
  alertSettingsDialogVisible.value = false
  ElMessage.success('告警设置已保存')
}

// 保存数据安全配置
const saveDataSecurityConfig = () => {
  ElMessage.success('数据安全配置已保存')
}

// 保存认证安全配置
const saveAuthSecurityConfig = () => {
  ElMessage.success('认证安全配置已保存')
}

// 保存日志安全配置
const saveLogSecurityConfig = () => {
  ElMessage.success('日志安全配置已保存')
}

// 保存所有配置
const saveAllConfigs = () => {
  ElMessage.success('所有配置已保存')
}

// 获取告警级别文本
const getAlertLevelText = (level) => {
  const map = {
    danger: '危险',
    warning: '警告',
    info: '信息'
  }
  return map[level] || level
}

// 获取告警类型文本
const getAlertTypeText = (type) => {
  const map = {
    brute_force: '暴力破解',
    sql_injection: 'SQL注入',
    xss: '跨站脚本',
    ddos: 'DDoS攻击',
    anomaly: '异常访问'
  }
  return map[type] || type
}

// 获取告警状态文本
const getAlertStatusText = (status) => {
  const map = {
    pending: '未处理',
    processed: '已处理',
    ignored: '已忽略'
  }
  return map[status] || status
}

// 获取告警状态标签类型
const getAlertStatusType = (status) => {
  const map = {
    pending: 'danger',
    processed: 'success',
    ignored: 'info'
  }
  return map[status] || 'info'
}

// 获取日志类型文本
const getLogTypeText = (type) => {
  const map = {
    login: '登录日志',
    operation: '操作日志',
    security: '安全日志',
    system: '系统日志'
  }
  return map[type] || type
}

// 获取日志类型标签
const getLogTypeTag = (type) => {
  const map = {
    login: 'info',
    operation: 'success',
    security: 'danger',
    system: 'warning'
  }
  return map[type] || 'info'
}

// 分页大小变化
const handleAlertSizeChange = (newSize) => {
  alertPageSize.value = newSize
  alertCurrentPage.value = 1
}

// 当前页码变化
const handleAlertCurrentChange = (newPage) => {
  alertCurrentPage.value = newPage
}

// 日志分页大小变化
const handleLogSizeChange = (newSize) => {
  logPageSize.value = newSize
  logCurrentPage.value = 1
}

// 日志当前页码变化
const handleLogCurrentChange = (newPage) => {
  logCurrentPage.value = newPage
}

// 生命周期钩子
onMounted(() => {
  initSecurityTrendChart()
  initSecurityDistributionChart()
})

onUnmounted(() => {
  // 清理图表实例
  if (securityTrendChartInstance) {
    securityTrendChartInstance.destroy()
  }
  if (securityDistributionChartInstance) {
    securityDistributionChartInstance.destroy()
  }
})
</script>

<style scoped>
.security-container {
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

.security-score-section {
  margin-bottom: 20px;
}

.score-card {
  padding: 20px;
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
  border-radius: var(--el-border-radius-large);
}

.score-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.score-title {
  font-size: 18px;
  font-weight: 600;
  color: var(--el-text-color-primary);
  margin: 0;
}

.score-subtitle {
  font-size: 14px;
  color: var(--el-text-color-secondary);
  margin: 4px 0 0 0;
}

.score-circle {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--el-color-success), var(--el-color-primary));
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: 700;
  font-size: 32px;
  position: relative;
}

.score-value {
  font-size: 48px;
}

.score-unit {
  font-size: 18px;
  margin-left: 4px;
  opacity: 0.8;
}

.score-metrics {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  padding-top: 20px;
  border-top: 1px solid var(--el-border-color);
}

.metric-item {
  text-align: center;
}

.metric-value {
  font-size: 24px;
  font-weight: 700;
  color: var(--el-text-color-primary);
  margin-bottom: 4px;
}

.metric-label {
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

.chart-desc {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.chart-container {
  height: 300px;
  padding: 10px;
}

.event-types {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  padding: 20px 0;
}

.event-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  background-color: var(--el-bg-color-page);
  border-radius: var(--el-border-radius-base);
}

.event-name {
  font-size: 14px;
  color: var(--el-text-color-primary);
}

.event-count {
  font-size: 16px;
  font-weight: 600;
  color: var(--el-color-primary);
}

.security-config-section {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  margin-bottom: 20px;
}

.config-card {
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
  margin-bottom: 20px;
}

.config-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16px;
  padding: 20px 0;
}

.config-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  background-color: var(--el-bg-color-page);
  border-radius: var(--el-border-radius-base);
  transition: all 0.3s ease;
}

.config-item:hover {
  transform: translateY(-2px);
  box-shadow: var(--el-box-shadow-light);
}

.config-icon {
  width: 48px;
  height: 48px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: bold;
}

.config-icon.success {
  background-color: rgba(72, 187, 120, 0.1);
  color: var(--el-color-success);
}

.config-icon.warning {
  background-color: rgba(237, 137, 54, 0.1);
  color: var(--el-color-warning);
}

.config-info {
  flex: 1;
}

.config-name {
  font-size: 14px;
  font-weight: 500;
  color: var(--el-text-color-primary);
  margin-bottom: 4px;
}

.config-status {
  font-size: 12px;
  color: var(--el-text-color-secondary);
}

.card-action {
  font-size: 14px;
  color: var(--el-color-primary);
  cursor: pointer;
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

.alerts-card,
.logs-card {
  background-color: var(--el-bg-color);
  box-shadow: var(--el-box-shadow-light);
}

.table-container {
  margin-bottom: 20px;
  overflow-x: auto;
}

.pagination-container {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  padding-top: 20px;
  border-top: 1px solid var(--el-border-color);
}

.config-form {
  padding: 20px 0;
}

.unit {
  margin-left: 8px;
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 16px;
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .score-metrics {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .grid-2 {
    grid-template-columns: 1fr;
  }
  
  .security-config-section {
    grid-template-columns: 1fr;
  }
  
  .config-grid {
    grid-template-columns: 1fr;
  }
  
  .event-types {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .score-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 20px;
  }
  
  .score-metrics {
    grid-template-columns: 1fr;
  }
  
  .top-actions {
    flex-direction: column;
    align-items: stretch;
  }
  
  .event-types {
    grid-template-columns: 1fr;
  }
  
  .form-row {
    flex-direction: column;
  }
  
  .form-row :deep(.el-form-item) {
    margin-right: 0;
    width: 100%;
  }
  
  .form-actions {
    flex-direction: column;
    align-items: stretch;
  }
  
  .pagination-container {
    flex-direction: column;
    align-items: flex-start;
    gap: 16px;
  }
}
</style>