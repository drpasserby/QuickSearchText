<template>
  <div class="container">
    <!-- 主输入框 -->
    <div class="main-input-wrapper">
      <textarea class="main-input"  v-model="mainText" placeholder="请输入主文本">
      </textarea>
    </div>

    <!-- 小输入框列表 -->
    <div class="tags-wrapper">
      <button class="user-btn toggle-btn" @click="showTags = !showTags">
          {{ showTags ? '隐藏关键词' : '显示关键词' }}
      </button>
      <div class="tag-input" v-show="showTags" v-for="(tag, index) in tags" :key="index">
        <div class="color-box" :style="{ backgroundColor: tag.color }" @click="removeTag(index)"></div>
        <input v-model="tag.value" placeholder="关键词"/>
        <button class="remove-tag" @click="removeTag(index)">×</button>
      </div>

      <button class="user-btn add-btn" v-show="showTags" @click="addTag">＋ 添加</button>
    </div>

    <!-- 关键词统计 -->
    <div class="instructions">
      <h3>关键词统计</h3>
      <ul>
        <li v-for="(count, keyword) in keywordCounts" :key="keyword">
          {{ keyword }}: {{ count }}
        </li>
      </ul>
    </div>
    <br>
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
const showTags = ref(true)

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
    const color = tag ? tag.color : '#ff0'
    return `<span style="background-color:${color};color:#fff;padding:0 2px;border-radius:2px;">${match}</span>`
  })
})

const keywordCounts = computed(() => {
  const counts = {}
  tags.value.forEach(tag => {
    if (tag.value.trim()) {
      const regex = new RegExp(tag.value.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'), 'gi')
      const matches = mainText.value.match(regex)
      counts[tag.value] = matches ? matches.length : 0
    }
  })
  return counts
})
</script>

<style>
.container {
  max-width: 800px;
  margin: 40px auto;
  padding: 20px 30px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,.1);
  transition: all .5s;
}

.container:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, .5);
}

.main-input-wrapper {
  text-align: center;
  margin-bottom: 20px;
  padding: 0 .5em;
}

.main-input {
  width: 90%;
  max-width: 600px;
  min-height: 4em;
  height: auto; 
  padding: 10px 14px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  transition: border-color 0.2s, box-shadow 0.2s;
  resize: vertical; 
  font-family: inherit;
}
.main-input:focus {
  border-color: #3498db;
  box-shadow: 0 0 4px rgba(52,152,219,.3);
  outline: none;
}

.tags-wrapper {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}

.tag-input {
  display: flex;
  align-items: center;
  background: #f9f9f9;
  padding: 6px 8px;
  border-radius: 6px;
  box-shadow: 0 1px 3px rgba(0,0,0,.1);
  position: relative;
}

.color-box {
  width: 20px;
  height: 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
  flex-shrink: 0;
}

.tag-input input {
  width: 100px;
  padding: 4px 6px;
  font-size: 14px;
  margin-left: 6px;
  border: none;
  outline: none;
  background: transparent;
}

.remove-tag {
  background: transparent;
  border: none;
  font-size: 16px;
  line-height: 1;
  padding: 0 4px;
  margin-left: 4px;
  cursor: pointer;
  color: #999;
}
.remove-tag:hover {
  color: #e74c3c;
}

.user-btn {
  padding: 6px 12px;
  font-size: 14px;
  cursor: pointer;
  color: #fff;
  border: none;
  border-radius: 4px;
  transition: background 0.2s;
  flex-basis: 100%;
  display: block;
  width: fit-content;
  margin: 6px auto 0;
}

.toggle-btn {
  background: #ecf0f1;
  color: #333;
}

.toggle-btn:hover{
  background: #b7b9b9;
}

.add-btn{
  background: #3498db;
}
.add-btn:hover {
  background: #2980b9;
}

.result-box {
  min-height: 120px;
  padding: 12px 16px;
  border: 1px solid #ddd;
  border-radius: 6px;
  white-space: pre-wrap;
  background: #fafafa;
  line-height: 1.5;
}
.instructions {
  margin-top: 20px;
  font-size: 14px;
  color: #555;
  background: #f5f5f5;
  padding: 12px 16px;
  border-radius: 6px;
  border: 1px solid #eee;
}
</style>
