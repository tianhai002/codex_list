<script setup>
import { computed, nextTick, ref } from 'vue';

const storageKey = 'qingdan.tasks.v1';
const savedMessage = '清单保存在当前浏览器';
const storageNote = ref(savedMessage);
const tasks = ref([]);
const filter = ref('all');
const draft = ref('');
const input = ref(null);
const list = ref(null);
let nextId = 0;

const filters = [
  { value: 'all', label: '全部', empty: '清单还是空的，添加第一件事吧。' },
  { value: 'open', label: '待完成', empty: '没有待完成的任务。' },
  { value: 'done', label: '已完成', empty: '还没有已完成的任务。' }
];
const visible = computed(() => tasks.value.filter(task =>
  filter.value === 'all' || task.done === (filter.value === 'done')
));
const remaining = computed(() => tasks.value.filter(task => !task.done).length);
const emptyMessage = computed(() => filters.find(option => option.value === filter.value).empty);

try {
  const saved = JSON.parse(localStorage.getItem(storageKey));
  if (Array.isArray(saved)) {
    tasks.value = saved
      .filter(task => typeof task?.title === 'string' && typeof task.done === 'boolean')
      .map(({ title, done }) => ({ id: nextId++, title, done }));
  }
} catch {
  storageNote.value = '暂时无法读取已保存的清单';
}

function save() {
  try {
    const records = tasks.value.map(({ title, done }) => ({ title, done }));
    localStorage.setItem(storageKey, JSON.stringify(records));
    storageNote.value = savedMessage;
  } catch {
    storageNote.value = '保存失败，刷新后可能丢失本次修改';
  }
}

function addTask() {
  const title = draft.value.trim();
  if (title) {
    tasks.value.unshift({ id: nextId++, title, done: false });
    save();
    draft.value = '';
    if (filter.value === 'done') filter.value = 'open';
  }
  input.value.focus();
}

async function focusRow(index, control) {
  await nextTick();
  const row = list.value.children[Math.min(index, visible.value.length - 1)];
  (row?.querySelector(control) || input.value).focus();
}

function toggleTask(task, done, index) {
  task.done = done;
  save();
  focusRow(index, 'input');
}

function removeTask(task, index) {
  tasks.value.splice(tasks.value.indexOf(task), 1);
  save();
  focusRow(index, 'button');
}
</script>

<template>
  <main>
    <header>
      <div class="eyebrow">MY TASKS / 个人待办</div>
      <h1>今天要做的事</h1>
      <p>把想到的事记下来，一件一件完成。</p>
    </header>

    <section class="card" aria-label="待办事项">
      <form id="add" @submit.prevent="addTask">
        <input
          id="task"
          ref="input"
          v-model="draft"
          aria-label="新任务"
          placeholder="写下一件要做的事…"
          required
          maxlength="200"
        >
        <button type="submit">＋ 添加</button>
      </form>

      <div class="toolbar">
        <nav aria-label="筛选任务">
          <button
            v-for="option in filters"
            :key="option.value"
            type="button"
            :data-filter="option.value"
            :aria-pressed="filter === option.value"
            @click="filter = option.value"
          >{{ option.label }}</button>
        </nav>
        <span id="count" role="status">剩余 {{ remaining }} 项</span>
      </div>

      <ul id="list" ref="list">
        <li v-for="(task, index) in visible" :key="task.id" :class="{ done: task.done }">
          <label>
            <input type="checkbox" :checked="task.done" @change="toggleTask(task, $event.target.checked, index)">
            <span>{{ task.title }}</span>
          </label>
          <button type="button" class="remove" :aria-label="'删除 ' + task.title" @click="removeTask(task, index)">
            删除
          </button>
        </li>
      </ul>
      <p v-if="!visible.length" id="empty">{{ emptyMessage }}</p>
    </section>

    <footer id="storage-note">{{ storageNote }}</footer>
  </main>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #f4f5ef;
  color: #23332a;
  font: 16px/1.5 system-ui,sans-serif;
}

main {
  width: min(680px,calc(100% - 32px));
  margin: clamp(32px,9vh,96px) auto;
}

header {
  background: #203d31;
  color: #fff;
  border-radius: 24px;
  padding: clamp(24px,6vw,42px);
}

.eyebrow {
  color: #d0e49b;
  font-size: 12px;
  letter-spacing: .18em;
}

h1 {
  font-size: clamp(36px,8vw,56px);
  line-height: 1.1;
  margin: 14px 0;
}

header p {
  color: #deeadf;
  margin-bottom: 0;
}

.card {
  margin-top: 16px;
  background: #fff;
  border: 1px solid #e1e7dc;
  border-radius: 24px;
  padding: clamp(16px,4vw,28px);
}

form,.toolbar,li,label {
  display: flex;
  align-items: center;
  gap: 10px;
}

form input {
  min-width: 0;
  flex: 1;
  border: 1px solid #c8d1c6;
  border-radius: 10px;
  padding: 12px;
  font: inherit;
}

button {
  cursor: pointer;
  font: inherit;
  border: 0;
  border-radius: 10px;
  padding: 10px 14px;
  color: #284336;
  background: #edf2e9;
}

form button,button[aria-pressed=true] {
  background: #315f47;
  color: white;
}

button:focus-visible,input:focus-visible {
  outline: 3px solid #759346;
  outline-offset: 2px;
}

.toolbar {
  justify-content: space-between;
  flex-wrap: wrap;
  margin-top: 22px;
}

nav {
  display: flex;
  gap: 5px;
}

nav button {
  padding: 6px 10px;
}

#count {
  font-size: 14px;
  color: #506455;
}

ul {
  list-style: none;
  margin: 14px 0 0;
  padding: 0;
}

li {
  border-top: 1px solid #e7ece5;
  justify-content: space-between;
  padding: 12px 0;
}

label {
  flex: 1;
  min-width: 0;
  cursor: pointer;
}

label span {
  overflow-wrap: anywhere;
}

input[type=checkbox] {
  width: 19px;
  height: 19px;
  flex: none;
  accent-color: #315f47;
}

.done span {
  text-decoration: line-through;
  color: #627367;
}

.remove {
  flex: none;
  font-size: 14px;
}

#empty {
  text-align: center;
  color: #506455;
  padding: 24px 0;
  margin: 0;
}

footer {
  text-align: center;
  color: #506455;
  font-size: 13px;
  margin-top: 20px;
}
</style>
