<script setup>
defineProps({ task: { type: Object, required: true } });
defineEmits(['toggle', 'delete']);
</script>

<template>
    <li class="item" :class="{ finished: task.done }">
        <button class="check" :class="{ on: task.done }" @click="$emit('toggle', task.id)" aria-label="Toggle task">
            <span v-if="task.done">&#10003;</span>
        </button>

       
        <span class="title" :class="{ done: task.done }">{{ task.title }}</span>

        
        <span
            class="badge"
            :class="task.priority.toLowerCase()"
        >{{ task.priority }}</span>

        <button class="del" @click="$emit('delete', task.id)" aria-label="Delete task">&#10005;</button>
    </li>
</template>

<style scoped>
.item { display: flex; align-items: center; gap: 12px; padding: 12px 14px;
        background: #f8fafc; border-radius: 12px; margin-bottom: 10px;
        border: 1px solid #eef2f7; transition: box-shadow .2s, transform .2s; }
.item:hover { box-shadow: 0 4px 14px rgba(99,102,241,.15); transform: translateY(-1px); }
.item.finished { opacity: .75; }

.check { width: 24px; height: 24px; border-radius: 50%; border: 2px solid #cbd5e1;
         background: #fff; cursor: pointer; display: grid; place-items: center;
         color: #fff; font-size: 13px; flex-shrink: 0; transition: all .2s; padding: 0; }
.check:hover { border-color: #6366f1; }
.check.on { background: #22c55e; border-color: #22c55e; }

.title { flex: 1; color: #1e293b; font-size: 15px; word-break: break-word; }
.title.done { text-decoration: line-through; color: #94a3b8; }

.badge { font-size: 11px; font-weight: 600; padding: 3px 10px; border-radius: 999px; }
.badge.high   { background: #fee2e2; color: #b91c1c; }
.badge.medium { background: #fef3c7; color: #b45309; }
.badge.low    { background: #dcfce7; color: #15803d; }

.del { border: 0; background: transparent; color: #94a3b8; cursor: pointer;
       font-size: 15px; padding: 4px 6px; border-radius: 6px; }
.del:hover { background: #fee2e2; color: #dc2626; }
</style>
