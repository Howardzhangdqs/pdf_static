<script setup lang="ts">
import { ref, computed } from 'vue'
import * as pdfjsLib from 'pdfjs-dist'
import type { TextItem } from 'pdfjs-dist/types/src/display/api'

// 使用本地 worker - 通过 Vite 的方式导入
import pdfWorker from 'pdfjs-dist/build/pdf.worker.min.mjs?url'
pdfjsLib.GlobalWorkerOptions.workerSrc = pdfWorker

// 统计结果类型
interface StatsResult {
  chineseChars: number
  englishWords: number
  englishLetters: number
  numbers: number
  chinesePunctuation: number
  englishPunctuation: number
  spaces: number
  lineBreaks: number
  totalChars: number
  totalCharsNoSpace: number
}

const isDragging = ref<boolean>(false)
const isLoading = ref<boolean>(false)
const fileName = ref<string>('')
const pdfText = ref<string>('')
const errorMessage = ref<string>('')

// 统计结果
const stats = computed<StatsResult | null>(() => {
  if (!pdfText.value) return null
  
  const text = pdfText.value
  
  // 中文汉字（包括扩展区）
  const chineseChars = text.match(/[\u4e00-\u9fff\u3400-\u4dbf\u20000-\u2a6df\u2a700-\u2b73f\u2b740-\u2b81f\u2b820-\u2ceaf\uf900-\ufaff\u2f800-\u2fa1f]/g) || []
  
  // 英文单词（连续的英文字母）
  const englishWords = text.match(/[a-zA-Z]+/g) || []
  
  // 英文字母总数
  const englishLetters = text.match(/[a-zA-Z]/g) || []
  
  // 数字
  const numbers = text.match(/[0-9]/g) || []
  
  // 中文标点
  const chinesePunctuation = text.match(/[，。！？、；：""''（）【】《》…—～·]/g) || []
  
  // 英文标点
  const englishPunctuation = text.match(/[,.!?;:'"()\[\]{}<>\/\\@#$%^&*\-_+=|`~]/g) || []
  
  // 空格
  const spaces = text.match(/\s/g) || []
  
  // 换行符
  const lineBreaks = text.match(/\n/g) || []
  
  // 总字符数（不含空格）
  const totalCharsNoSpace = text.replace(/\s/g, '').length
  
  return {
    chineseChars: chineseChars.length,
    englishWords: englishWords.length,
    englishLetters: englishLetters.length,
    numbers: numbers.length,
    chinesePunctuation: chinesePunctuation.length,
    englishPunctuation: englishPunctuation.length,
    spaces: spaces.length,
    lineBreaks: lineBreaks.length,
    totalChars: text.length,
    totalCharsNoSpace
  }
})

// 处理文件上传
const handleFile = async (file: File): Promise<void> => {
  if (!file || file.type !== 'application/pdf') {
    errorMessage.value = '请上传 PDF 文件'
    return
  }
  
  errorMessage.value = ''
  isLoading.value = true
  fileName.value = file.name
  pdfText.value = ''
  
  try {
    const arrayBuffer = await file.arrayBuffer()
    const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise
    
    let fullText = ''
    
    for (let i = 1; i <= pdf.numPages; i++) {
      const page = await pdf.getPage(i)
      const textContent = await page.getTextContent()
      const pageText = textContent.items
        .filter((item): item is TextItem => 'str' in item)
        .map(item => item.str)
        .join('')
      fullText += pageText + '\n'
    }
    
    pdfText.value = fullText
  } catch (error) {
    console.error('PDF 解析错误:', error)
    errorMessage.value = 'PDF 解析失败，请确保文件未损坏'
  } finally {
    isLoading.value = false
  }
}

// 文件输入处理
const onFileInput = (event: Event): void => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (file) handleFile(file)
}

// 拖拽处理
const onDragOver = (event: DragEvent): void => {
  event.preventDefault()
  isDragging.value = true
}

const onDragLeave = (): void => {
  isDragging.value = false
}

const onDrop = (event: DragEvent): void => {
  event.preventDefault()
  isDragging.value = false
  const file = event.dataTransfer?.files[0]
  if (file) handleFile(file)
}

// 重置
const reset = (): void => {
  fileName.value = ''
  pdfText.value = ''
  errorMessage.value = ''
}
</script>

<template>
  <div class="app">
    <!-- 头部 -->
    <header class="header">
      <div class="logo">
        <div class="logo-icon">📊</div>
        <h1>PDF 统计</h1>
      </div>
      <p class="header-desc">快速分析 PDF 文本内容</p>
    </header>

    <!-- 主体内容 -->
    <main class="main">
      <!-- 上传区域 -->
      <section class="upload-section">
        <div
          class="upload-zone"
          :class="{
            'upload-zone--dragging': isDragging,
            'upload-zone--has-file': fileName,
            'upload-zone--loading': isLoading
          }"
          @dragover="onDragOver"
          @dragleave="onDragLeave"
          @drop="onDrop"
        >
          <input
            type="file"
            accept=".pdf,application/pdf"
            @change="onFileInput"
            id="file-input"
            class="upload-input"
          />

          <!-- 加载状态 -->
          <div v-if="isLoading" class="upload-state">
            <div class="loading-spinner"></div>
            <p class="upload-text">解析中...</p>
          </div>

          <!-- 未选择文件 -->
          <div v-else-if="!fileName" class="upload-state">
            <div class="upload-icon">📁</div>
            <p class="upload-text">拖拽 PDF 文件</p>
            <label for="file-input" class="upload-button">选择文件</label>
          </div>

          <!-- 已选择文件 -->
          <div v-else class="upload-state">
            <div class="file-icon">📄</div>
            <p class="file-name">{{ fileName }}</p>
            <button @click="reset" class="reset-button">更换</button>
          </div>
        </div>

        <!-- 错误提示 -->
        <div v-if="errorMessage" class="error">
          {{ errorMessage }}
        </div>
      </section>

      <!-- 统计结果 -->
      <section v-if="stats" class="stats-section">
        <div class="stats-header">
          <h2>统计结果</h2>
        </div>

        <!-- 主要统计 -->
        <div class="stats-main">
          <div class="stat-item">
            <div class="stat-number">{{ stats.chineseChars.toLocaleString() }}</div>
            <div class="stat-label">中文字符</div>
          </div>
          <div class="stat-item">
            <div class="stat-number">{{ stats.englishWords.toLocaleString() }}</div>
            <div class="stat-label">英文单词</div>
          </div>
          <div class="stat-item">
            <div class="stat-number">{{ stats.numbers.toLocaleString() }}</div>
            <div class="stat-label">数字</div>
          </div>
          <div class="stat-item">
            <div class="stat-number">{{ stats.totalCharsNoSpace.toLocaleString() }}</div>
            <div class="stat-label">总字符</div>
          </div>
        </div>

        <!-- 详细统计 -->
        <div class="stats-details">
          <div class="detail-group">
            <div class="detail-row">
              <span>英文字母</span>
              <span>{{ stats.englishLetters.toLocaleString() }}</span>
            </div>
            <div class="detail-row">
              <span>中文标点</span>
              <span>{{ stats.chinesePunctuation.toLocaleString() }}</span>
            </div>
            <div class="detail-row">
              <span>英文标点</span>
              <span>{{ stats.englishPunctuation.toLocaleString() }}</span>
            </div>
          </div>

          <div class="detail-group">
            <div class="detail-row">
              <span>空格字符</span>
              <span>{{ stats.spaces.toLocaleString() }}</span>
            </div>
            <div class="detail-row">
              <span>换行符</span>
              <span>{{ stats.lineBreaks.toLocaleString() }}</span>
            </div>
            <div class="detail-row">
              <span>包含空格总计</span>
              <span class="detail-total">{{ stats.totalChars.toLocaleString() }}</span>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* 基础设置 */
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  background: #f5f7fa;
  color: #1a202c;
  line-height: 1.6;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 头部 */
.header {
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  padding: 24px 20px;
  text-align: center;
}

.logo {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 4px;
}

.logo-icon {
  font-size: 28px;
  line-height: 1;
}

.header h1 {
  font-size: 24px;
  font-weight: 600;
  color: #1a202c;
}

.header-desc {
  color: #64748b;
  font-size: 14px;
}

/* 主体 */
.main {
  flex: 1;
  padding: 32px 20px;
  max-width: 600px;
  margin: 0 auto;
  width: 100%;
}

/* 上传区域 */
.upload-section {
  margin-bottom: 48px;
}

.upload-zone {
  background: #ffffff;
  border: 2px dashed #cbd5e1;
  border-radius: 12px;
  padding: 48px 24px;
  text-align: center;
  transition: all 0.2s ease;
  cursor: pointer;
}

.upload-zone:hover {
  border-color: #6366f1;
  background: #f8f9ff;
}

.upload-zone--dragging {
  border-color: #6366f1;
  background: #eef2ff;
}

.upload-zone--has-file {
  border-style: solid;
  border-color: #10b981;
}

.upload-zone--loading {
  border-color: #fbbf24;
  background: #fefdf8;
}

.upload-input {
  display: none;
}

.upload-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.upload-icon {
  font-size: 48px;
  opacity: 0.8;
}

.file-icon {
  font-size: 40px;
}

.upload-text {
  color: #475569;
  font-size: 16px;
  font-weight: 500;
}

.upload-button {
  display: inline-flex;
  align-items: center;
  padding: 8px 20px;
  background: #6366f1;
  color: white;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s ease;
}

.upload-button:hover {
  background: #4f46e5;
}

.file-name {
  color: #1a202c;
  font-size: 15px;
  font-weight: 500;
  max-width: 300px;
  word-break: break-all;
}

.reset-button {
  padding: 6px 16px;
  background: #f1f5f9;
  color: #475569;
  border: none;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.reset-button:hover {
  background: #e2e8f0;
  color: #334155;
}

.loading-spinner {
  width: 32px;
  height: 32px;
  border: 3px solid #f3f4f6;
  border-top: 3px solid #6366f1;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error {
  margin-top: 16px;
  padding: 12px 16px;
  background: #fef2f2;
  color: #dc2626;
  border-radius: 8px;
  font-size: 14px;
  border-left: 3px solid #dc2626;
}

/* 统计结果 */
.stats-section {
  background: #ffffff;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.stats-header {
  margin-bottom: 24px;
}

.stats-header h2 {
  font-size: 20px;
  font-weight: 600;
  color: #1a202c;
}

/* 主要统计 */
.stats-main {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 20px;
  margin-bottom: 32px;
}

.stat-item {
  text-align: center;
  padding: 20px 16px;
  background: #f8fafc;
  border-radius: 8px;
  border: 1px solid #e2e8f0;
}

.stat-number {
  font-size: 28px;
  font-weight: 700;
  color: #1a202c;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 13px;
  color: #64748b;
  font-weight: 500;
}

/* 详细统计 */
.stats-details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 32px;
  padding-top: 24px;
  border-top: 1px solid #e2e8f0;
}

.detail-group {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  font-size: 14px;
}

.detail-row span:first-child {
  color: #64748b;
}

.detail-row span:last-child {
  color: #1a202c;
  font-weight: 500;
}

.detail-total {
  color: #6366f1 !important;
  font-weight: 600 !important;
}

/* 响应式设计 */
@media (max-width: 640px) {
  .main {
    padding: 24px 16px;
  }

  .stats-main {
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
  }

  .stat-item {
    padding: 16px 12px;
  }

  .stat-number {
    font-size: 24px;
  }

  .stats-details {
    grid-template-columns: 1fr;
    gap: 24px;
  }

  .upload-zone {
    padding: 36px 20px;
  }

  .upload-icon {
    font-size: 40px;
  }

  .file-icon {
    font-size: 32px;
  }
}

/* 动画效果 */
.upload-zone {
  transition: border-color 0.2s ease, background-color 0.2s ease;
}

.stat-item {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
</style>
