<template>
  <div class="container">
    <!-- 主输入框 -->
    <div class="main-input-wrapper">
      <input class="main-input"  v-model="mainText" placeholder="请输入主文本"/>
    </div>

    <!-- 小输入框列表 -->
    <div class="tags-wrapper">
      <div class="tag-input"  v-for="(tag, index) in tags" :key="index">
        <div class="color-box" :style="{ backgroundColor: tag.color }" @click="removeTag(index)"></div>
        <input v-model="tag.value" placeholder="关键词"/>
      </div>
      <button class="add-btn" @click="addTag">添加</button>
    </div>
    
    <!-- 显示结果 -->
    <div class="result-box" v-html="highlightedText"></div>

    <!-- 使用说明 -->
    <div class="instructions">
      <p>在上方主输入框输入要搜索的文本，在下方关键词框中添加多个词条并为其选择颜色。系统会自动高亮匹配的关键词。</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const mainText = ref('')

// 初始几个小输入框，预置不同颜色
const tags = ref([
  { value: '', color: '#e74c3c' },
  { value: '', color: '#27ae60' },
  { value: '', color: '#2980b9' },
])

function addTag() {
  tags.value.push({ value: '', color: getRandomColor() })
}

function removeTag(idx) {
  tags.value.splice(idx, 1)
}

function getRandomColor() {
  // 生成随机颜色
  return '#' + Math.floor(Math.random() * 0xffffff).toString(16).padStart(6, '0')
}

const highlightedText = computed(() => {
  const text = mainText.value
  if (!text) return ''

  const patterns = tags.value
    .map(t => t.value.trim())
    .filter(v => v)
    .map(v => v.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'))

  if (patterns.length === 0) return text

  const regex = new RegExp(`(${patterns.join('|')})`, 'gi')
  return text.replace(regex, match => {
    const tag = tags.value.find(
      t => t.value && t.value.toLowerCase() === match.toLowerCase()
    )
    const color = tag ? tag.color : 'yellow'
    return `<span style="color:${color}">${match}</span>`
  })
})
</script>

<style scoped>
.container {
  padding: 20px;
  font-family: sans-serif;
}

.main-input-wrapper {
  text-align: center;
  margin-bottom: 20px;
}

.main-input {
  width: 60%;
  padding: 8px 12px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.tags-wrapper {
  text-align: center;
  margin-bottom: 20px;
}

.tag-input {
  display: inline-block;
  margin: 0 10px 10px;
  text-align: center;
  position: relative;
}

.color-box {
  width: 20px;
  height: 20px;
  margin: 0 auto 4px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.tag-input input {
  width: 100px;
  padding: 4px;
  font-size: 14px;
  box-sizing: border-box;
}

.add-btn {
  padding: 4px 8px;
  font-size: 14px;
  cursor: pointer;
}

.result-box {
  min-height: 100px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  white-space: pre-wrap;
}
</style>
