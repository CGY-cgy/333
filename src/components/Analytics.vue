<!--
数据分析组件
API配置信息：
- 开发环境：http://localhost:8080/api/v1
- 生产环境：https://api.lingjing.com/v1
- 请求超时时间：30秒
对应API接口：
- GET /api/analytics/overview - 获取数据概览
- GET /api/analytics/users - 获取用户分析数据
- GET /api/analytics/content - 获取内容分析数据
- GET /api/analytics/ai - 获取AI分析数据
- GET /api/analytics/reports - 获取报表列表
- POST /api/analytics/reports - 创建报表
- GET /api/analytics/reports/:id - 获取报表详情
- GET /api/analytics/reports/:id/export - 导出报表
-->
<template>
  <div class="analytics-container">
    <!-- 页面标题和操作 -->
    <div class="header-section">
      <h1 class="page-title">数据分析</h1>
      <p class="page-subtitle">平台功能使用数据与用户行为分析</p>
    </div>

    <!-- 标签页导航 -->
    <el-tabs v-model="activeTab" class="analytics-tabs" @tab-change="handleTabChange">
      <!-- 数据概览 -->
      <el-tab-pane label="数据概览" name="overview">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-select v-model="currentDateRange" placeholder="选择日期范围" size="large" @change="updateAllCharts">
            <el-option label="最近7天" :value="7" />
            <el-option label="最近30天" :value="30" />
            <el-option label="最近90天" :value="90" />
          </el-select>
          <el-button type="primary" size="large" @click="exportData">
            <el-icon-document-download />
            <span>导出报表</span>
          </el-button>
        </div>

        <!-- 统计数据网格 -->
        <div class="stats-grid">
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon visits">👁️</div>
              <span class="stat-trend up">↑ 15.3%</span>
            </div>
            <div class="stat-value">{{ formatNumber(1234567) }}</div>
            <div class="stat-label">总访问量</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon active">🔥</div>
              <span class="stat-trend up">↑ 8.7%</span>
            </div>
            <div class="stat-value">{{ formatNumber(45892) }}</div>
            <div class="stat-label">日活跃用户</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon duration">⏱️</div>
              <span class="stat-trend up">↑ 5.2%</span>
            </div>
            <div class="stat-value">12.5 分钟</div>
            <div class="stat-label">平均使用时长</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon retention">🔄</div>
              <span class="stat-trend up">↑ 3.1%</span>
            </div>
            <div class="stat-value">67.8%</div>
            <div class="stat-label">次日留存率</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon completion">✅</div>
              <span class="stat-trend down">↓ 1.2%</span>
            </div>
            <div class="stat-value">89.3%</div>
            <div class="stat-label">功能完成率</div>
          </el-card>
        </div>

        <!-- 功能使用热力图 -->
        <el-card shadow="hover" class="chart-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">功能使用热力图</h3>
              <span class="chart-desc">各核心模块的使用频率</span>
            </div>
          </template>
          <div class="heatmap-section">
            <div class="heatmap-info">过去 {{ currentDateRange }} 天各模块使用频率</div>
            <div ref="heatmap" class="heatmap-container"></div>
            <div class="heatmap-weekdays">
              <span>周一</span><span>周二</span><span>周三</span><span>周四</span><span>周五</span><span>周六</span><span>周日</span>
            </div>
            <div class="heatmap-legend">
              <span>使用频率：</span>
              <div class="legend-colors">
                <span class="legend-item">
                  <span class="legend-color low"></span>
                  <span class="legend-text">低</span>
                </span>
                <span class="legend-item">
                  <span class="legend-color medium"></span>
                  <span class="legend-text">中</span>
                </span>
                <span class="legend-item">
                  <span class="legend-color high"></span>
                  <span class="legend-text">高</span>
                </span>
              </div>
            </div>
          </div>
        </el-card>
      </el-tab-pane>

      <!-- 用户分析 -->
      <el-tab-pane label="用户分析" name="user-analysis">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-select v-model="currentDateRange" placeholder="选择日期范围" size="large" @change="updateAllCharts">
            <el-option label="最近7天" :value="7" />
            <el-option label="最近30天" :value="30" />
            <el-option label="最近90天" :value="90" />
          </el-select>
        </div>

        <!-- 统计数据网格 -->
        <div class="stats-grid">
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon visits">👥</div>
              <span class="stat-trend up">↑ 12.5%</span>
            </div>
            <div class="stat-value">{{ formatNumber(123456) }}</div>
            <div class="stat-label">总用户数</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon active">✨</div>
              <span class="stat-trend up">↑ 8.3%</span>
            </div>
            <div class="stat-value">{{ formatNumber(45678) }}</div>
            <div class="stat-label">活跃用户数</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon duration">📈</div>
              <span class="stat-trend up">↑ 5.7%</span>
            </div>
            <div class="stat-value">{{ formatNumber(12345) }}</div>
            <div class="stat-label">新增用户数</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon retention">🔄</div>
              <span class="stat-trend up">↑ 2.1%</span>
            </div>
            <div class="stat-value">67.8%</div>
            <div class="stat-label">用户留存率</div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- 用户活跃度分布 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户活跃度分布</h3>
                <span class="chart-desc">不同活跃度用户占比</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="userActivityChart"></canvas>
            </div>
          </el-card>

          <!-- 用户增长趋势 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户增长趋势</h3>
                <span class="chart-desc">新用户注册与活跃用户变化</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="userGrowthChart"></canvas>
            </div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- 用户留存分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户留存分析</h3>
                <span class="chart-desc">不同时间段用户留存率</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="userRetentionChart"></canvas>
            </div>
          </el-card>

          <!-- 用户设备分布 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户设备分布</h3>
                <span class="chart-desc">用户使用设备类型占比</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="userDeviceChart"></canvas>
            </div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- 用户地域分布 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户地域分布</h3>
                <span class="chart-desc">用户地理位置分布</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="userRegionChart"></canvas>
            </div>
          </el-card>

          <!-- 用户行为漏斗 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户行为漏斗</h3>
                <span class="chart-desc">用户从访问到转化的流程</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="userFunnelChart"></canvas>
            </div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- 用户使用时长分布 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户使用时长分布</h3>
                <span class="chart-desc">用户每次使用平台的时长分布</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="userDurationChart"></canvas>
            </div>
          </el-card>

          <!-- 用户功能使用偏好 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">用户功能使用偏好</h3>
                <span class="chart-desc">用户对不同功能的使用频率</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="userFeatureChart"></canvas>
            </div>
          </el-card>
        </div>
      </el-tab-pane>

      <!-- 内容分析 -->
      <el-tab-pane label="内容分析" name="content-analysis">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-select v-model="currentDateRange" placeholder="选择日期范围" size="large" @change="updateAllCharts">
            <el-option label="最近7天" :value="7" />
            <el-option label="最近30天" :value="30" />
            <el-option label="最近90天" :value="90" />
          </el-select>
        </div>

        <!-- 统计数据网格 -->
        <div class="stats-grid">
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon visits">📝</div>
              <span class="stat-trend up">↑ 15.8%</span>
            </div>
            <div class="stat-value">{{ formatNumber(12345) }}</div>
            <div class="stat-label">总内容量</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon active">🔥</div>
              <span class="stat-trend up">↑ 12.3%</span>
            </div>
            <div class="stat-value">{{ formatNumber(67890) }}</div>
            <div class="stat-label">总互动量</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon duration">📈</div>
              <span class="stat-trend up">↑ 8.7%</span>
            </div>
            <div class="stat-value">{{ formatNumber(1234) }}</div>
            <div class="stat-label">日均发帖量</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon retention">💡</div>
              <span class="stat-trend up">↑ 3.5%</span>
            </div>
            <div class="stat-value">5.2</div>
            <div class="stat-label">平均互动率</div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- 内容生态分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">内容生态分析</h3>
                <div class="chart-legend">
                  <div class="legend-item">
                    <span class="legend-color primary"></span>
                    <span class="legend-text">发帖量</span>
                  </div>
                  <div class="legend-item">
                    <span class="legend-color success"></span>
                    <span class="legend-text">互动量</span>
                  </div>
                  <div class="legend-item">
                    <span class="legend-color warning"></span>
                    <span class="legend-text">互动率</span>
                  </div>
                </div>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="contentEcosystemChart"></canvas>
            </div>
          </el-card>

          <!-- 内容类型分布 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">内容类型分布</h3>
                <span class="chart-desc">不同类型内容占比</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="contentTypeChart"></canvas>
            </div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- 内容热度趋势 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">内容热度趋势</h3>
                <span class="chart-desc">内容互动热度变化</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="contentHeatChart"></canvas>
            </div>
          </el-card>

          <!-- 内容发布时间分布 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">内容发布时间分布</h3>
                <span class="chart-desc">用户发布内容的时间规律</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="contentTimeChart"></canvas>
            </div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- 内容质量分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">内容质量分析</h3>
                <span class="chart-desc">不同质量等级内容占比</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="contentQualityChart"></canvas>
            </div>
          </el-card>

          <!-- 内容互动分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">内容互动分析</h3>
                <span class="chart-desc">不同互动类型占比</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="contentInteractionChart"></canvas>
            </div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- 内容标签分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">内容标签分析</h3>
                <span class="chart-desc">热门标签使用频率</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="contentTagChart"></canvas>
            </div>
          </el-card>

          <!-- 内容传播分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">内容传播分析</h3>
                <span class="chart-desc">内容分享与传播效果</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="contentShareChart"></canvas>
            </div>
          </el-card>
        </div>
      </el-tab-pane>

      <!-- AI分析 -->
      <el-tab-pane label="AI分析" name="ai-analysis">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-select v-model="currentDateRange" placeholder="选择日期范围" size="large" @change="updateAllCharts">
            <el-option label="最近7天" :value="7" />
            <el-option label="最近30天" :value="30" />
            <el-option label="最近90天" :value="90" />
          </el-select>
        </div>

        <!-- 统计数据网格 -->
        <div class="stats-grid">
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon visits">🤖</div>
              <span class="stat-trend up">↑ 25.6%</span>
            </div>
            <div class="stat-value">{{ formatNumber(12345) }}</div>
            <div class="stat-label">总对话数</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon active">⚡</div>
              <span class="stat-trend down">↓ 12.3%</span>
            </div>
            <div class="stat-value">3.2s</div>
            <div class="stat-label">平均响应时间</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon duration">😊</div>
              <span class="stat-trend up">↑ 1.8%</span>
            </div>
            <div class="stat-value">96.7%</div>
            <div class="stat-label">用户满意度</div>
          </el-card>
          <el-card shadow="hover" class="stat-card">
            <div class="stat-header">
              <div class="stat-icon retention">✅</div>
              <span class="stat-trend up">↑ 3.5%</span>
            </div>
            <div class="stat-value">89.2%</div>
            <div class="stat-label">问题解决率</div>
          </el-card>
        </div>

        <!-- AI对话分析 -->
        <el-card shadow="hover" class="chart-card ai-chat-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">AI对话分析</h3>
              <span class="chart-desc">AI助手使用情况统计</span>
            </div>
          </template>
          <div class="ai-chat-stats">
            <div class="ai-stat-item">
              <div class="ai-stat-value">1,234</div>
              <div class="ai-stat-label">今日对话数</div>
            </div>
            <div class="ai-stat-item">
              <div class="ai-stat-value">3.2s</div>
              <div class="ai-stat-label">平均回复时长</div>
            </div>
            <div class="ai-stat-item">
              <div class="ai-stat-value">96.7%</div>
              <div class="ai-stat-label">用户满意度</div>
            </div>
            <div class="ai-stat-item">
              <div class="ai-stat-value">89.2%</div>
              <div class="ai-stat-label">问题解决率</div>
            </div>
          </div>
          <div class="chart-container">
            <canvas ref="aiChatChart"></canvas>
          </div>
        </el-card>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- AI功能使用分布 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">AI功能使用分布</h3>
                <span class="chart-desc">不同AI功能的使用频率</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="aiFunctionChart"></canvas>
            </div>
          </el-card>

          <!-- AI响应时间分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">AI响应时间分析</h3>
                <span class="chart-desc">AI回复时间分布</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="aiResponseTimeChart"></canvas>
            </div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- AI解决率分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">AI解决率分析</h3>
                <span class="chart-desc">不同类型问题的解决率</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="aiResolutionChart"></canvas>
            </div>
          </el-card>

          <!-- AI对话主题分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">AI对话主题分析</h3>
                <span class="chart-desc">用户咨询的主题分布</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="aiTopicChart"></canvas>
            </div>
          </el-card>
        </div>

        <!-- 图表网格 -->
        <div class="charts-grid">
          <!-- AI用户使用频率分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">AI用户使用频率分析</h3>
                <span class="chart-desc">用户使用AI助手的频率分布</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="aiUsageFrequencyChart"></canvas>
            </div>
          </el-card>

          <!-- AI对话长度分析 -->
          <el-card shadow="hover" class="chart-card">
            <template #header>
              <div class="card-header-content">
                <h3 class="card-title">AI对话长度分析</h3>
                <span class="chart-desc">对话轮数分布</span>
              </div>
            </template>
            <div class="chart-container">
              <canvas ref="aiConversationLengthChart"></canvas>
            </div>
          </el-card>
        </div>
      </el-tab-pane>

      <!-- 报表管理 -->
      <el-tab-pane label="报表管理" name="report-management">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="refreshReports">
            <el-icon-refresh />
            <span>刷新报表</span>
          </el-button>
          <el-button type="success" @click="exportData">
            <el-icon-document-download />
            <span>导出报表</span>
          </el-button>
        </div>

        <!-- 报表列表 -->
        <el-table :data="reports" style="width: 100%" border stripe>
          <el-table-column prop="id" label="报表ID" width="80" />
          <el-table-column prop="name" label="报表名称" />
          <el-table-column prop="type" label="报表类型" width="120">
            <template #default="scope">
              <span v-if="scope.row.type === 'overview'">数据概览</span>
              <span v-else-if="scope.row.type === 'user'">用户分析</span>
              <span v-else-if="scope.row.type === 'content'">内容分析</span>
              <span v-else-if="scope.row.type === 'ai'">AI分析</span>
            </template>
          </el-table-column>
          <el-table-column prop="createTime" label="创建时间" width="180" />
          <el-table-column prop="status" label="状态" width="100">
            <template #default="scope">
              <el-tag :type="scope.row.status === 'generated' ? 'success' : 'warning'">
                {{ scope.row.status === 'generated' ? '已生成' : '生成中' }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="180" fixed="right">
            <template #default="scope">
              <el-button type="primary" size="small" @click="viewReport(scope.row.id)">查看</el-button>
              <el-button type="warning" size="small" @click="downloadReport(scope.row.id)">下载</el-button>
              <el-button type="danger" size="small" @click="deleteReport(scope.row.id)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>

        <!-- 分页 -->
        <div class="pagination">
          <el-pagination
            v-model:current-page="reportCurrentPage"
            v-model:page-size="reportPageSize"
            :page-sizes="[10, 20, 50, 100]"
            layout="total, sizes, prev, pager, next, jumper"
            :total="reportsTotal"
            @size-change="handleReportSizeChange"
            @current-change="handleReportCurrentChange"
          />
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { Chart, registerables } from 'chart.js'
import * as XLSX from 'xlsx'
import { ElMessage } from 'element-plus'
import {
  Download,
  Refresh,
  Search
} from '@element-plus/icons-vue'

// 注册Chart.js组件
Chart.register(...registerables)

// 定义组件别名
const ElIconDocumentDownload = Download
const ElIconRefresh = Refresh
const ElIconSearch = Search

// 标签页状态
const activeTab = ref('overview')

// 响应式数据
const currentDateRange = ref(30)

// 报表管理数据
const reports = ref([
  { id: 1, name: '数据概览报表', type: 'overview', createTime: '2026-02-03 10:00:00', status: 'generated' },
  { id: 2, name: '用户分析报表', type: 'user', createTime: '2026-02-03 09:30:00', status: 'generated' },
  { id: 3, name: '内容分析报表', type: 'content', createTime: '2026-02-03 09:00:00', status: 'generated' },
  { id: 4, name: 'AI分析报表', type: 'ai', createTime: '2026-02-02 18:00:00', status: 'generated' },
  { id: 5, name: '数据概览报表', type: 'overview', createTime: '2026-02-02 10:00:00', status: 'generated' },
  { id: 6, name: '用户分析报表', type: 'user', createTime: '2026-02-02 09:30:00', status: 'generated' },
  { id: 7, name: '内容分析报表', type: 'content', createTime: '2026-02-02 09:00:00', status: 'generated' },
  { id: 8, name: 'AI分析报表', type: 'ai', createTime: '2026-02-01 18:00:00', status: 'generated' },
  { id: 9, name: '数据概览报表', type: 'overview', createTime: '2026-02-01 10:00:00', status: 'generated' },
  { id: 10, name: '用户分析报表', type: 'user', createTime: '2026-02-01 09:30:00', status: 'generated' }
])
const reportCurrentPage = ref(1)
const reportPageSize = ref(10)
const reportsTotal = ref(50)

// 图表引用
const heatmap = ref(null)
const userActivityChart = ref(null)
const userGrowthChart = ref(null)
const userRetentionChart = ref(null)
const userDeviceChart = ref(null)
const userRegionChart = ref(null)
const userFunnelChart = ref(null)
const userDurationChart = ref(null)
const userFeatureChart = ref(null)
const contentEcosystemChart = ref(null)
const contentTypeChart = ref(null)
const contentHeatChart = ref(null)
const contentTimeChart = ref(null)
const contentQualityChart = ref(null)
const contentInteractionChart = ref(null)
const contentTagChart = ref(null)
const contentShareChart = ref(null)
const aiChatChart = ref(null)
const aiFunctionChart = ref(null)
const aiResponseTimeChart = ref(null)
const aiResolutionChart = ref(null)
const aiTopicChart = ref(null)
const aiUsageFrequencyChart = ref(null)
const aiConversationLengthChart = ref(null)

// 图表实例
let userActivityChartInstance = null
let userGrowthChartInstance = null
let userRetentionChartInstance = null
let userDeviceChartInstance = null
let userRegionChartInstance = null
let userFunnelChartInstance = null
let userDurationChartInstance = null
let userFeatureChartInstance = null
let contentEcosystemChartInstance = null
let contentTypeChartInstance = null
let contentHeatChartInstance = null
let contentTimeChartInstance = null
let contentQualityChartInstance = null
let contentInteractionChartInstance = null
let contentTagChartInstance = null
let contentShareChartInstance = null
let aiChatChartInstance = null
let aiFunctionChartInstance = null
let aiResponseTimeChartInstance = null
let aiResolutionChartInstance = null
let aiTopicChartInstance = null
let aiUsageFrequencyChartInstance = null
let aiConversationLengthChartInstance = null

// 颜色配置
const chartColors = {
  primary: '#667eea',
  secondary: '#764ba2',
  success: '#48bb78',
  warning: '#ed8936',
  danger: '#f56565',
  info: '#4299e1',
  light: '#f7fafc',
  dark: '#2d3748'
}

// 格式化数字
const formatNumber = (num) => {
  return num.toLocaleString()
}

// 报表管理相关方法
const refreshReports = () => {
  // 实现刷新逻辑
  ElMessage.success('报表列表已刷新')
}

const viewReport = (id) => {
  // 实现查看报表逻辑
  ElMessage.success(`查看报表 ${id}`)
}

const downloadReport = (id) => {
  // 实现下载报表逻辑
  ElMessage.success(`下载报表 ${id}`)
}

const deleteReport = (id) => {
  // 实现删除报表逻辑
  ElMessage.success(`删除报表 ${id}`)
}

const handleReportSizeChange = (size) => {
  reportPageSize.value = size
}

const handleReportCurrentChange = (current) => {
  reportCurrentPage.value = current
}

// 标签页切换处理
const handleTabChange = (tab) => {
  activeTab.value = tab
  // 确保图表初始化
  initAllCharts()
}

// 初始化所有图表
const initAllCharts = () => {
  initHeatmap()
  initUserActivityChart()
  initUserGrowthChart()
  initUserRetentionChart()
  initUserDeviceChart()
  initUserRegionChart()
  initUserFunnelChart()
  initUserDurationChart()
  initUserFeatureChart()
  initContentEcosystemChart()
  initContentTypeChart()
  initContentHeatChart()
  initContentTimeChart()
  initContentQualityChart()
  initContentInteractionChart()
  initContentTagChart()
  initContentShareChart()
  initAIChatChart()
  initAIFunctionChart()
  initAIResponseTimeChart()
  initAIResolutionChart()
  initAITopicChart()
  initAIUsageFrequencyChart()
  initAIConversationLengthChart()
}

// 初始化热力图
const initHeatmap = () => {
  if (heatmap.value) {
    heatmap.value.innerHTML = ''
    const days = currentDateRange.value
    const modules = ['命理', '健康', '饮食', '规划', 'AI']
    
    for (let i = 0; i < days; i++) {
      const cell = document.createElement('div')
      cell.className = 'heatmap-cell'
      const level = Math.floor(Math.random() * 5) + 1 // 1-5级
      cell.classList.add(`level-${level}`)
      cell.title = `${modules[i % 5]}模块 - 第${i + 1}天`
      heatmap.value.appendChild(cell)
    }
  }
}

// 初始化用户活跃度分布图表
const initUserActivityChart = () => {
  if (userActivityChart.value) {
    if (userActivityChartInstance) {
      userActivityChartInstance.destroy()
    }
    const ctx = userActivityChart.value.getContext('2d')
    
    userActivityChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['低活跃度', '中活跃度', '高活跃度', '非常活跃'],
        datasets: [{
          label: '用户数量',
          data: [35000, 68000, 25000, 12000],
          backgroundColor: [
            'rgba(245, 101, 101, 0.8)',
            'rgba(237, 137, 54, 0.8)',
            'rgba(72, 187, 120, 0.8)',
            'rgba(102, 126, 234, 0.8)'
          ],
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

// 初始化用户增长趋势图表
const initUserGrowthChart = () => {
  if (userGrowthChart.value) {
    if (userGrowthChartInstance) {
      userGrowthChartInstance.destroy()
    }
    const ctx = userGrowthChart.value.getContext('2d')
    
    let labels, newUsers, activeUsers
    if (currentDateRange.value === 7) {
      labels = ['1日', '2日', '3日', '4日', '5日', '6日', '7日']
      newUsers = [120, 150, 180, 160, 200, 230, 210]
      activeUsers = [850, 920, 980, 950, 1050, 1120, 1080]
    } else if (currentDateRange.value === 30) {
      labels = ['1日', '5日', '10日', '15日', '20日', '25日', '30日']
      newUsers = [3500, 4200, 4800, 4500, 5200, 5800, 5500]
      activeUsers = [28000, 31000, 34000, 33000, 36000, 39000, 38000]
    } else {
      labels = ['1月', '2月', '3月', '4月', '5月', '6月', '7月']
      newUsers = [12000, 14500, 16800, 15900, 18200, 20500, 19800]
      activeUsers = [95000, 105000, 115000, 112000, 120000, 128000, 126000]
    }
    
    userGrowthChartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [{
          label: '新用户',
          data: newUsers,
          borderColor: chartColors.primary,
          backgroundColor: 'rgba(102, 126, 234, 0.1)',
          fill: true,
          tension: 0.4
        }, {
          label: '活跃用户',
          data: activeUsers,
          borderColor: chartColors.success,
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
            position: 'top'
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

// 初始化用户留存分析图表
const initUserRetentionChart = () => {
  if (userRetentionChart.value) {
    if (userRetentionChartInstance) {
      userRetentionChartInstance.destroy()
    }
    const ctx = userRetentionChart.value.getContext('2d')
    
    userRetentionChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['次日', '3日', '7日', '14日', '30日'],
        datasets: [{
          label: '留存率',
          data: [67.8, 52.3, 41.5, 32.7, 25.1],
          backgroundColor: chartColors.primary,
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
            max: 100,
            grid: {
              color: 'rgba(0, 0, 0, 0.05)'
            },
            ticks: {
              callback: function(value) {
                return value + '%'
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

// 初始化用户设备分布图表
const initUserDeviceChart = () => {
  if (userDeviceChart.value) {
    if (userDeviceChartInstance) {
      userDeviceChartInstance.destroy()
    }
    const ctx = userDeviceChart.value.getContext('2d')
    
    userDeviceChartInstance = new Chart(ctx, {
      type: 'doughnut',
      data: {
        labels: ['移动端', 'PC端', '平板', '其他'],
        datasets: [{
          data: [65, 25, 8, 2],
          backgroundColor: [
            chartColors.primary,
            chartColors.success,
            chartColors.warning,
            chartColors.danger
          ],
          borderWidth: 2,
          borderColor: '#fff'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            position: 'right'
          }
        }
      }
    })
  }
}

// 初始化用户地域分布图表
const initUserRegionChart = () => {
  if (userRegionChart.value) {
    if (userRegionChartInstance) {
      userRegionChartInstance.destroy()
    }
    const ctx = userRegionChart.value.getContext('2d')
    
    userRegionChartInstance = new Chart(ctx, {
      type: 'pie',
      data: {
        labels: ['北京', '上海', '广州', '深圳', '杭州', '成都', '其他'],
        datasets: [{
          data: [18, 15, 12, 10, 8, 7, 30],
          backgroundColor: [
            chartColors.primary,
            chartColors.success,
            chartColors.warning,
            chartColors.danger,
            chartColors.info,
            chartColors.secondary,
            '#e2e8f0'
          ],
          borderWidth: 2,
          borderColor: '#fff'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            position: 'right'
          }
        }
      }
    })
  }
}

// 初始化用户行为漏斗图表
const initUserFunnelChart = () => {
  if (userFunnelChart.value) {
    if (userFunnelChartInstance) {
      userFunnelChartInstance.destroy()
    }
    const ctx = userFunnelChart.value.getContext('2d')
    
    userFunnelChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['访问', '注册', '浏览', '互动', '转化'],
        datasets: [{
          label: '用户数量',
          data: [100000, 65000, 45000, 25000, 12000],
          backgroundColor: [
            'rgba(102, 126, 234, 0.8)',
            'rgba(118, 75, 162, 0.8)',
            'rgba(72, 187, 120, 0.8)',
            'rgba(237, 137, 54, 0.8)',
            'rgba(245, 101, 101, 0.8)'
          ],
          borderRadius: 8,
          borderSkipped: false
        }]
      },
      options: {
        indexAxis: 'y',
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            display: false
          }
        },
        scales: {
          x: {
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
          y: {
            grid: {
              display: false
            }
          }
        }
      }
    })
  }
}

// 初始化用户使用时长分布图表
const initUserDurationChart = () => {
  if (userDurationChart.value) {
    if (userDurationChartInstance) {
      userDurationChartInstance.destroy()
    }
    const ctx = userDurationChart.value.getContext('2d')
    
    userDurationChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['0-5分钟', '5-15分钟', '15-30分钟', '30-60分钟', '60分钟以上'],
        datasets: [{
          label: '用户数量',
          data: [15000, 28000, 18000, 8000, 3000],
          backgroundColor: [
            'rgba(245, 101, 101, 0.8)',
            'rgba(237, 137, 54, 0.8)',
            'rgba(72, 187, 120, 0.8)',
            'rgba(102, 126, 234, 0.8)',
            'rgba(118, 75, 162, 0.8)'
          ],
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

// 初始化用户功能使用偏好图表
const initUserFeatureChart = () => {
  if (userFeatureChart.value) {
    if (userFeatureChartInstance) {
      userFeatureChartInstance.destroy()
    }
    const ctx = userFeatureChart.value.getContext('2d')
    
    userFeatureChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['命理分析', '健康建议', '饮食规划', '生活规划', 'AI对话', '社区互动'],
        datasets: [{
          label: '使用频率',
          data: [35000, 28000, 22000, 18000, 32000, 25000],
          backgroundColor: chartColors.primary,
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

// 初始化内容生态分析图表
const initContentEcosystemChart = () => {
  if (contentEcosystemChart.value) {
    if (contentEcosystemChartInstance) {
      contentEcosystemChartInstance.destroy()
    }
    const ctx = contentEcosystemChart.value.getContext('2d')
    
    let labels, postData, interactionData, rateData
    if (currentDateRange.value === 7) {
      labels = ['1日', '2日', '3日', '4日', '5日', '6日', '7日']
      postData = [156, 189, 234, 198, 267, 312, 289]
      interactionData = [892, 1056, 1234, 1189, 1456, 1876, 1623]
      rateData = [5.7, 5.6, 5.3, 6.0, 5.4, 6.0, 5.6]
    } else if (currentDateRange.value === 30) {
      labels = ['1日', '5日', '10日', '15日', '20日', '25日', '30日']
      postData = [4200, 4800, 5100, 4900, 5600, 6200, 5800]
      interactionData = [21560, 24890, 26540, 25120, 28760, 32150, 30480]
      rateData = [5.1, 5.2, 5.2, 5.1, 5.1, 5.2, 5.2]
    } else {
      labels = ['1月', '2月', '3月', '4月', '5月', '6月', '7月']
      postData = [12500, 14200, 15300, 14800, 16700, 18500, 17600]
      interactionData = [62890, 71450, 76230, 73890, 82450, 91230, 87650]
      rateData = [5.0, 5.0, 5.0, 4.9, 4.9, 4.9, 4.9]
    }
    
    contentEcosystemChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: labels,
        datasets: [{
          label: '发帖量',
          data: postData,
          backgroundColor: chartColors.primary,
          borderRadius: 8,
          borderSkipped: false,
          yAxisID: 'y'
        }, {
          label: '互动量',
          data: interactionData,
          backgroundColor: chartColors.success,
          borderRadius: 8,
          borderSkipped: false,
          yAxisID: 'y'
        }, {
          label: '互动率(%)',
          data: rateData,
          type: 'line',
          borderColor: chartColors.warning,
          backgroundColor: 'transparent',
          borderWidth: 2,
          borderDash: [5, 5],
          tension: 0.4,
          yAxisID: 'y1'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            position: 'top'
          }
        },
        scales: {
          y: {
            type: 'linear',
            position: 'left',
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
          y1: {
            type: 'linear',
            position: 'right',
            beginAtZero: true,
            max: 10,
            grid: {
              display: false
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

// 初始化内容类型分布图表
const initContentTypeChart = () => {
  if (contentTypeChart.value) {
    if (contentTypeChartInstance) {
      contentTypeChartInstance.destroy()
    }
    const ctx = contentTypeChart.value.getContext('2d')
    
    contentTypeChartInstance = new Chart(ctx, {
      type: 'doughnut',
      data: {
        labels: ['文章', '图片', '视频', '问答', '评论'],
        datasets: [{
          data: [35, 25, 20, 15, 5],
          backgroundColor: [
            chartColors.primary,
            chartColors.success,
            chartColors.warning,
            chartColors.danger,
            chartColors.info
          ],
          borderWidth: 2,
          borderColor: '#fff'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            position: 'right'
          }
        }
      }
    })
  }
}

// 初始化内容热度趋势图表
const initContentHeatChart = () => {
  if (contentHeatChart.value) {
    if (contentHeatChartInstance) {
      contentHeatChartInstance.destroy()
    }
    const ctx = contentHeatChart.value.getContext('2d')
    
    let labels, heatData
    if (currentDateRange.value === 7) {
      labels = ['1日', '2日', '3日', '4日', '5日', '6日', '7日']
      heatData = [85, 92, 88, 95, 98, 105, 100]
    } else if (currentDateRange.value === 30) {
      labels = ['1日', '5日', '10日', '15日', '20日', '25日', '30日']
      heatData = [280, 320, 350, 330, 380, 410, 390]
    } else {
      labels = ['1月', '2月', '3月', '4月', '5月', '6月', '7月']
      heatData = [1200, 1350, 1480, 1420, 1580, 1720, 1650]
    }
    
    contentHeatChartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [{
          label: '内容热度指数',
          data: heatData,
          borderColor: chartColors.warning,
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
            position: 'top'
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            grid: {
              color: 'rgba(0, 0, 0, 0.05)'
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

// 初始化内容发布时间分布图表
const initContentTimeChart = () => {
  if (contentTimeChart.value) {
    if (contentTimeChartInstance) {
      contentTimeChartInstance.destroy()
    }
    const ctx = contentTimeChart.value.getContext('2d')
    
    contentTimeChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['00:00', '04:00', '08:00', '12:00', '16:00', '20:00'],
        datasets: [{
          label: '发布量',
          data: [50, 30, 120, 180, 220, 280],
          backgroundColor: chartColors.primary,
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

// 初始化内容质量分析图表
const initContentQualityChart = () => {
  if (contentQualityChart.value) {
    if (contentQualityChartInstance) {
      contentQualityChartInstance.destroy()
    }
    const ctx = contentQualityChart.value.getContext('2d')
    
    contentQualityChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['优质', '良好', '一般', '较差', '差'],
        datasets: [{
          label: '内容数量',
          data: [1200, 2800, 3500, 1800, 700],
          backgroundColor: [
            chartColors.success,
            chartColors.info,
            chartColors.warning,
            chartColors.danger,
            '#f56565'
          ],
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

// 初始化内容互动分析图表
const initContentInteractionChart = () => {
  if (contentInteractionChart.value) {
    if (contentInteractionChartInstance) {
      contentInteractionChartInstance.destroy()
    }
    const ctx = contentInteractionChart.value.getContext('2d')
    
    contentInteractionChartInstance = new Chart(ctx, {
      type: 'pie',
      data: {
        labels: ['点赞', '评论', '分享', '收藏', '其他'],
        datasets: [{
          data: [45, 25, 15, 10, 5],
          backgroundColor: [
            chartColors.primary,
            chartColors.success,
            chartColors.warning,
            chartColors.danger,
            chartColors.info
          ],
          borderWidth: 2,
          borderColor: '#fff'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            position: 'right'
          }
        }
      }
    })
  }
}

// 初始化内容标签分析图表
const initContentTagChart = () => {
  if (contentTagChart.value) {
    if (contentTagChartInstance) {
      contentTagChartInstance.destroy()
    }
    const ctx = contentTagChart.value.getContext('2d')
    
    contentTagChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['健康', '饮食', '生活', '工作', '学习', '娱乐', '其他'],
        datasets: [{
          label: '使用频率',
          data: [3500, 2800, 2200, 1800, 1500, 1200, 800],
          backgroundColor: chartColors.primary,
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

// 初始化内容传播分析图表
const initContentShareChart = () => {
  if (contentShareChart.value) {
    if (contentShareChartInstance) {
      contentShareChartInstance.destroy()
    }
    const ctx = contentShareChart.value.getContext('2d')
    
    contentShareChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['微信', '微博', 'QQ', '知乎', '小红书', '其他'],
        datasets: [{
          label: '分享次数',
          data: [3500, 2800, 1800, 1500, 1200, 800],
          backgroundColor: chartColors.primary,
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

// 初始化AI对话分析图表
const initAIChatChart = () => {
  if (aiChatChart.value) {
    if (aiChatChartInstance) {
      aiChatChartInstance.destroy()
    }
    const ctx = aiChatChart.value.getContext('2d')
    
    let labels, chatData
    if (currentDateRange.value === 7) {
      labels = ['1日', '2日', '3日', '4日', '5日', '6日', '7日']
      chatData = [120, 150, 180, 160, 200, 230, 210]
    } else if (currentDateRange.value === 30) {
      labels = ['1日', '5日', '10日', '15日', '20日', '25日', '30日']
      chatData = [3500, 4200, 4800, 4500, 5200, 5800, 5500]
    } else {
      labels = ['1月', '2月', '3月', '4月', '5月', '6月', '7月']
      chatData = [12000, 14500, 16800, 15900, 18200, 20500, 19800]
    }
    
    aiChatChartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: [{
          label: '对话数',
          data: chatData,
          borderColor: chartColors.primary,
          backgroundColor: 'rgba(102, 126, 234, 0.1)',
          fill: true,
          tension: 0.4
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            position: 'top'
          }
        },
        scales: {
          y: {
            beginAtZero: true,
            grid: {
              color: 'rgba(0, 0, 0, 0.05)'
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

// 初始化AI功能使用分布图表
const initAIFunctionChart = () => {
  if (aiFunctionChart.value) {
    if (aiFunctionChartInstance) {
      aiFunctionChartInstance.destroy()
    }
    const ctx = aiFunctionChart.value.getContext('2d')
    
    aiFunctionChartInstance = new Chart(ctx, {
      type: 'doughnut',
      data: {
        labels: ['问答', '建议', '分析', '规划', '其他'],
        datasets: [{
          data: [40, 25, 20, 10, 5],
          backgroundColor: [
            chartColors.primary,
            chartColors.success,
            chartColors.warning,
            chartColors.danger,
            chartColors.info
          ],
          borderWidth: 2,
          borderColor: '#fff'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            position: 'right'
          }
        }
      }
    })
  }
}

// 初始化AI响应时间分析图表
const initAIResponseTimeChart = () => {
  if (aiResponseTimeChart.value) {
    if (aiResponseTimeChartInstance) {
      aiResponseTimeChartInstance.destroy()
    }
    const ctx = aiResponseTimeChart.value.getContext('2d')
    
    aiResponseTimeChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['<1s', '1-2s', '2-3s', '3-4s', '>4s'],
        datasets: [{
          label: '响应次数',
          data: [3500, 4200, 2800, 1200, 500],
          backgroundColor: [
            chartColors.success,
            chartColors.info,
            chartColors.warning,
            chartColors.danger,
            '#f56565'
          ],
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

// 初始化AI解决率分析图表
const initAIResolutionChart = () => {
  if (aiResolutionChart.value) {
    if (aiResolutionChartInstance) {
      aiResolutionChartInstance.destroy()
    }
    const ctx = aiResolutionChart.value.getContext('2d')
    
    aiResolutionChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['问答', '建议', '分析', '规划', '其他'],
        datasets: [{
          label: '解决率',
          data: [95.2, 92.8, 89.5, 87.3, 85.1],
          backgroundColor: chartColors.primary,
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
            max: 100,
            grid: {
              color: 'rgba(0, 0, 0, 0.05)'
            },
            ticks: {
              callback: function(value) {
                return value + '%'
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

// 初始化AI对话主题分析图表
const initAITopicChart = () => {
  if (aiTopicChart.value) {
    if (aiTopicChartInstance) {
      aiTopicChartInstance.destroy()
    }
    const ctx = aiTopicChart.value.getContext('2d')
    
    aiTopicChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['健康', '饮食', '生活', '工作', '学习', '娱乐', '其他'],
        datasets: [{
          label: '对话次数',
          data: [3500, 2800, 2200, 1800, 1500, 1200, 800],
          backgroundColor: chartColors.primary,
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

// 初始化AI用户使用频率分析图表
const initAIUsageFrequencyChart = () => {
  if (aiUsageFrequencyChart.value) {
    if (aiUsageFrequencyChartInstance) {
      aiUsageFrequencyChartInstance.destroy()
    }
    const ctx = aiUsageFrequencyChart.value.getContext('2d')
    
    aiUsageFrequencyChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['高频', '中频', '低频', '很少', '从未'],
        datasets: [{
          label: '用户数量',
          data: [1200, 2800, 3500, 1800, 700],
          backgroundColor: [
            chartColors.success,
            chartColors.info,
            chartColors.warning,
            chartColors.danger,
            '#f56565'
          ],
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

// 初始化AI对话长度分析图表
const initAIConversationLengthChart = () => {
  if (aiConversationLengthChart.value) {
    if (aiConversationLengthChartInstance) {
      aiConversationLengthChartInstance.destroy()
    }
    const ctx = aiConversationLengthChart.value.getContext('2d')
    
    aiConversationLengthChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['1-3轮', '4-6轮', '7-10轮', '11-15轮', '>15轮'],
        datasets: [{
          label: '对话次数',
          data: [3500, 2800, 1800, 1200, 700],
          backgroundColor: chartColors.primary,
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
  initAllCharts()
}

// 导出数据
const exportData = () => {
  // 实现数据导出逻辑
  ElMessage.success('数据导出功能已触发')
}

// 生命周期钩子
onMounted(() => {
  initAllCharts()
})

onUnmounted(() => {
  // 销毁图表实例
  if (userActivityChartInstance) userActivityChartInstance.destroy()
  if (userGrowthChartInstance) userGrowthChartInstance.destroy()
  if (userRetentionChartInstance) userRetentionChartInstance.destroy()
  if (userDeviceChartInstance) userDeviceChartInstance.destroy()
  if (userRegionChartInstance) userRegionChartInstance.destroy()
  if (userFunnelChartInstance) userFunnelChartInstance.destroy()
  if (userDurationChartInstance) userDurationChartInstance.destroy()
  if (userFeatureChartInstance) userFeatureChartInstance.destroy()
  if (contentEcosystemChartInstance) contentEcosystemChartInstance.destroy()
  if (contentTypeChartInstance) contentTypeChartInstance.destroy()
  if (contentHeatChartInstance) contentHeatChartInstance.destroy()
  if (contentTimeChartInstance) contentTimeChartInstance.destroy()
  if (contentQualityChartInstance) contentQualityChartInstance.destroy()
  if (contentInteractionChartInstance) contentInteractionChartInstance.destroy()
  if (contentTagChartInstance) contentTagChartInstance.destroy()
  if (contentShareChartInstance) contentShareChartInstance.destroy()
  if (aiChatChartInstance) aiChatChartInstance.destroy()
  if (aiFunctionChartInstance) aiFunctionChartInstance.destroy()
  if (aiResponseTimeChartInstance) aiResponseTimeChartInstance.destroy()
  if (aiResolutionChartInstance) aiResolutionChartInstance.destroy()
  if (aiTopicChartInstance) aiTopicChartInstance.destroy()
  if (aiUsageFrequencyChartInstance) aiUsageFrequencyChartInstance.destroy()
  if (aiConversationLengthChartInstance) aiConversationLengthChartInstance.destroy()
})
</script>

<style scoped>
.analytics-container {
  padding: 20px;
  background-color: #f5f7fa;
  min-height: 100vh;
}

.header-section {
  margin-bottom: 30px;
  text-align: center;
}

.page-title {
  font-size: 32px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 10px;
}

.page-subtitle {
  font-size: 16px;
  color: #606266;
}

.analytics-tabs {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.top-actions {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  align-items: center;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.stat-card {
  transition: all 0.3s ease;
}

.stat-card:hover {
  transform: translateY(-5px);
}

.stat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.stat-icon {
  font-size: 24px;
}

.stat-trend {
  font-size: 14px;
  font-weight: bold;
}

.stat-trend.up {
  color: #67c23a;
}

.stat-trend.down {
  color: #f56c6c;
}

.stat-value {
  font-size: 28px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 5px;
}

.stat-label {
  font-size: 14px;
  color: #606266;
}

.chart-card {
  margin-bottom: 30px;
  transition: all 0.3s ease;
}

.chart-card:hover {
  transform: translateY(-5px);
}

.card-header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-title {
  font-size: 18px;
  font-weight: bold;
  color: #303133;
  margin: 0;
}

.chart-desc {
  font-size: 14px;
  color: #606266;
}

.chart-container {
  height: 300px;
  margin-top: 20px;
}

.heatmap-section {
  margin-top: 20px;
}

.heatmap-info {
  font-size: 14px;
  color: #606266;
  margin-bottom: 15px;
}

.heatmap-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(30px, 1fr));
  gap: 5px;
  margin-bottom: 15px;
}

.heatmap-cell {
  width: 30px;
  height: 30px;
  border-radius: 4px;
  transition: all 0.3s ease;
}

.heatmap-cell:hover {
  transform: scale(1.1);
}

.heatmap-cell.level-1 { background-color: #e2e8f0; }
.heatmap-cell.level-2 { background-color: #cbd5e0; }
.heatmap-cell.level-3 { background-color: #94a3b8; }
.heatmap-cell.level-4 { background-color: #64748b; }
.heatmap-cell.level-5 { background-color: #475569; }

.heatmap-weekdays {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  font-size: 12px;
  color: #606266;
}

.heatmap-legend {
  display: flex;
  align-items: center;
  gap: 15px;
  font-size: 14px;
  color: #606266;
}

.legend-colors {
  display: flex;
  gap: 10px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 5px;
}

.legend-color {
  width: 16px;
  height: 16px;
  border-radius: 4px;
}

.legend-color.low { background-color: #e2e8f0; }
.legend-color.medium { background-color: #94a3b8; }
.legend-color.high { background-color: #475569; }

.legend-color.primary { background-color: #667eea; }
.legend-color.success { background-color: #48bb78; }
.legend-color.warning { background-color: #ed8936; }

.analysis-section {
  margin-bottom: 40px;
}

.section-title {
  font-size: 24px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 2px solid #e4e7ed;
}

.charts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 30px;
  margin-bottom: 30px;
}

.ai-chat-card {
  margin-bottom: 30px;
}

.ai-chat-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 20px;
  margin-bottom: 20px;
}

.ai-stat-item {
  text-align: center;
  padding: 15px;
  background-color: #f9fafb;
  border-radius: 8px;
}

.ai-stat-value {
  font-size: 24px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 5px;
}

.ai-stat-label {
  font-size: 14px;
  color: #606266;
}

.table-card {
  margin-bottom: 30px;
}

.content-preview {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}

.report-stats {
  margin-top: 30px;
}

.stats-title {
  font-size: 18px;
  font-weight: bold;
  color: #303133;
  margin-bottom: 20px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .top-actions {
    flex-direction: column;
    align-items: stretch;
  }

  .charts-grid {
    grid-template-columns: 1fr;
  }

  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .card-header-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }
}

@media (max-width: 480px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }

  .chart-container {
    height: 250px;
  }
}
</style>