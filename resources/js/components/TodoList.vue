<script setup>
import { ref, computed, watch } from 'vue';
import TodoItem from './TodoItem.vue';


const title = ref('');
const priority = ref('Medium');
const saved = localStorage.getItem('tasks');
const tasks = ref(saved ? JSON.parse(saved) : []);
const error = ref('');
const filter = ref('All');
watch(tasks, (val) => {
    localStorage.setItem('tasks', JSON.stringify(val));
}, { deep: true });
const search = ref('');


function addTask() {
    if (title.value.trim() === '') {
        error.value = 'Please type a task first.';
        return;
    }
    error.value = '';
    tasks.value.unshift({
        id: Date.now(),
        title: title.value.trim(),
        priority: priority.value,
        done: false,
    });
    title.value = '';
}

function toggleTask(id) {
    const t = tasks.value.find(t => t.id === id);
    if (t) t.done = !t.done;
}

function deleteTask(id) {
    tasks.value = tasks.value.filter(t => t.id !== id);
}

function clearDone() {
    tasks.value = tasks.value.filter(t => !t.done);
}


const doneCount = computed(() => tasks.value.filter(t => t.done).length);
const pendingCount = computed(() => tasks.value.length - doneCount.value);
const progress = computed(() =>
    tasks.value.length === 0 ? 0 : Math.round((doneCount.value / tasks.value.length) * 100)
);
const visibleTasks = computed(() => {
    let list = tasks.value;
    if (filter.value === 'Done') list = list.filter(t => t.done);
    if (filter.value === 'Pending') list = list.filter(t => !t.done);
    const q = search.value.trim().toLowerCase();
    if (q) list = list.filter(t => t.title.toLowerCase().includes(q));
    return list;
});

const sortBy = ref('Newest');
const priorityOrder = { High: 0, Medium: 1, Low: 2 };
const sortedTasks = computed(() => {
    const list = [...visibleTasks.value];
    if (sortBy.value === 'Priority') {
        list.sort((a, b) => priorityOrder[a.priority] - priorityOrder[b.priority]);
    }
    return list;
});

const today = new Date().toLocaleDateString('en-US', {
    weekday: 'long', month: 'long', day: 'numeric',
});


const message = ref('');
watch(progress, (now) => {
    message.value = now === 100 && tasks.value.length > 0
        ? 'All tasks completed. Great job!'
        : '';
});
</script>

<template>
    <div class="wrap">
        <div class="card">
            <header>
                <div>
                    <h1>My Tasks</h1>
                    <p class="date">{{ today }}</p>
                </div>
                <div class="count">
                    <strong>{{ pendingCount }}</strong>
                    <span>pending</span>
                </div>
            </header>

            <div class="progress">
                <div class="bar"><div class="fill" :style="{ width: progress + '%' }"></div></div>
                <small>{{ doneCount }} of {{ tasks.length }} done ({{ progress }}%)</small>
            </div>

            <div class="form">
                <input
                    v-model="title"
                    placeholder="What do you need to do?"
                    :class="{ invalid: error }"
                    @keyup.enter="addTask"
                />
                <select v-model="priority">
                    <option>Low</option>
                    <option>Medium</option>
                    <option>High</option>
                </select>
                <button class="add" @click="addTask">Add</button>
            </div>

            <p v-show="error" class="error">{{ error }}</p>

            <input v-model="search" class="search" placeholder="Search tasks..." />

            <div class="filters">
                <button
                    v-for="f in ['All', 'Pending', 'Done']"
                    :key="f"
                    :class="{ active: filter === f }"
                    @click="filter = f"
                >{{ f }}</button>
                <select v-model="sortBy" class="sort">
                    <option>Newest</option>
                    <option>Priority</option>
                </select>
            </div>

            <TransitionGroup v-if="visibleTasks.length > 0" name="list" tag="ul">
                <TodoItem
                    v-for="task in sortedTasks"
                    :key="task.id"
                    :task="task"
                    @toggle="toggleTask"
                    @delete="deleteTask"
                />
            </TransitionGroup>
            <div v-else class="empty">
                <div class="emoji">&#128221;</div>
                <p>No tasks to show.</p>
                <small>Add your first task above.</small>
            </div>

            <p v-if="message" class="success">{{ message }}</p>

            <footer v-if="doneCount > 0">
                <button class="clear" @click="clearDone">Clear completed</button>
            </footer>
        </div>
    </div>
</template>


<style>
body {
    margin: 0;
    min-height: 100vh;
    background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 50%, #ec4899 100%);
    font-family: 'Segoe UI', system-ui, sans-serif;
}
</style>

<style scoped>
.wrap { padding: 40px 16px; }
.card { max-width: 560px; margin: 0 auto; background: #fff; border-radius: 20px;
        padding: 28px; box-shadow: 0 20px 50px rgba(0,0,0,.25); }

header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 18px; }
h1 { margin: 0; font-size: 26px; color: #1e293b; }
.date { margin: 2px 0 0; color: #64748b; font-size: 14px; }
.count { text-align: center; background: #eef2ff; color: #4f46e5; padding: 8px 16px; border-radius: 12px; }
.count strong { display: block; font-size: 22px; line-height: 1; }
.count span { font-size: 12px; }

.progress { margin-bottom: 20px; }
.bar { height: 10px; background: #e2e8f0; border-radius: 999px; overflow: hidden; }
.fill { height: 100%; background: linear-gradient(90deg, #6366f1, #22c55e);
        border-radius: 999px; transition: width .4s ease; }
.progress small { color: #64748b; display: block; margin-top: 6px; }

.form { display: flex; gap: 8px; }
.form input { flex: 1; min-width: 0; padding: 12px 14px; border: 2px solid #e2e8f0;
              border-radius: 12px; font-size: 15px; outline: none; transition: border-color .2s; }
.form input:focus { border-color: #6366f1; }
.form input.invalid { border-color: #f87171; }
.form select { padding: 12px 10px; border: 2px solid #e2e8f0; border-radius: 12px;
               background: #fff; font-size: 14px; cursor: pointer; }
.add { background: linear-gradient(135deg, #6366f1, #8b5cf6); color: #fff; border: 0;
       padding: 0 22px; border-radius: 12px; font-weight: 600; cursor: pointer;
       transition: transform .15s, box-shadow .15s; }
.add:hover { transform: translateY(-1px); box-shadow: 0 6px 16px rgba(99,102,241,.4); }

.error { color: #dc2626; font-size: 14px; margin: 8px 2px 0; }
.success { color: #15803d; background: #dcfce7; padding: 10px 14px; border-radius: 10px;
           text-align: center; font-weight: 600; margin-top: 12px; }
.search { width: 100%; box-sizing: border-box; margin-top: 18px; padding: 10px 14px;
          border: 2px solid #e2e8f0; border-radius: 12px; font-size: 14px; outline: none; }
.search:focus { border-color: #6366f1; }

.filters { display: flex; gap: 8px; margin: 18px 0 14px; }
.filters button { border: 0; background: #f1f5f9; color: #475569; padding: 6px 16px;
                  border-radius: 999px; cursor: pointer; font-size: 13px; font-weight: 600; }
.filters button:hover { background: #e0e7ff; }
.filters .active { background: #6366f1; color: #fff; }

ul { list-style: none; padding: 0; margin: 0; }

.empty { text-align: center; padding: 28px 0 14px; color: #64748b; }
.empty .emoji { font-size: 40px; }
.empty p { margin: 6px 0 2px; font-weight: 600; }

footer { margin-top: 14px; text-align: right; }
.clear { background: transparent; border: 1px solid #cbd5e1; color: #64748b;
         padding: 6px 14px; border-radius: 10px; cursor: pointer; font-size: 13px; }
.clear:hover { background: #fee2e2; border-color: #fca5a5; color: #b91c1c; }
.sort { margin-left: auto; padding: 6px 10px; border: 2px solid #e2e8f0;
        border-radius: 10px; font-size: 13px; background: #fff; cursor: pointer; }
.sort:focus { border-color: #6366f1; outline: none; }

/* List animations */
.list-enter-active, .list-leave-active { transition: all .3s ease; }
.list-enter-from { opacity: 0; transform: translateY(-12px); }
.list-leave-to { opacity: 0; transform: translateX(30px); }
.list-move { transition: transform .3s ease; }
</style>