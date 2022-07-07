<template>
  <div>
    <transition name="modal">
      <div class="info-wrapper" v-if="showModal">
        <div class="info">哥，你啥也没输入！</div>
      </div>
    </transition>
    <input type="text" v-model="title" @keydown.enter="addTodo" />
    <button v-if="active < all" @click="clear">清理</button>
    <ul v-if="todos.length">
      <transition-group name="flip-list" tag="ul">
        <li v-for="(todo,i) in todos" :key="todo.id">
          <input type="checkbox" v-model="todo.done" />
          <span :class="{ done: todo.done }"> {{ todo.title }}</span>
          <span>id:{{todo.id}}</span>
          <button class="remove-btn" @click="removeTodo($event, i)"> 删除 </button>
        </li>
      </transition-group>
    </ul>
    <div v-else>暂无数据</div>
    <div>
      全选<input type="checkbox" v-model="allDone" />
      <span> {{ active }} / {{ all }} </span>
    </div>

    <span class="dustbin"> 🗑 </span>
    <div class="animate-wrap">
      <transition
        @before-enter="beforeEnter"
        @enter="enter"
        @after-enter="afterEnter"
      >
        <div class="animate" v-show="animate.show">📋</div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed,reactive } from 'vue'
let title = ref('')
let todos = ref([])
let showModal = ref(false)

function addTodo() {
  if (!title.value) {
    showModal.value = true
    setTimeout(() => {
      showModal.value = false
    }, 1500)
    return
  }
  todos.value.push({ title: title.value,id:todos.value.length+1, done: false })
  title.value = ''
}

let animate = reactive({ show: false, el: null })
function beforeEnter(el) {
  let dom = animate.el
  let rect = dom.getBoundingClientRect()
  let x = window.innerWidth - rect.left - 60
  let y = rect.top - 10
  el.style.transform = `translate(-${x}px, ${y}px)`
}

function enter(el, done) {
  document.body.offsetHeight
  el.style.transform = `translate(0,0)`
  el.addEventListener('transitionend', done)
}
function afterEnter(el) {
  animate.show = false
  el.style.display = 'none'
}
function removeTodo(e, i) {
  animate.el = e.target
  animate.show = true
  todos.value.splice(i, 1)
}

function clear() {
  todos.value = todos.value.filter((v) => !v.done)
}
let active = computed(() => {
  return todos.value.filter((v) => !v.done).length
})
let all = computed(() => todos.value.length)
let allDone = computed({
  get: function () {
    return active.value === 0
  },
  set: function (value) {
    todos.value.forEach((todo) => {
      todo.done = value
    })
  },
})
</script>
<style>
.flip-list-move {
  transition: transform 0.8s ease;
}
.flip-list-enter-active,
.flip-list-leave-active {
  transition: all 0.5s ease;
}
.flip-list-enter-from,
.flip-list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
.info-wrapper {
  position: fixed;
  top: 20px;
  width: 200px;
}
.info {
  padding: 20px;
  color: white;
  background: #d88986;
}
.modal-enter-from {
  opacity: 0;
  transform: translateY(-60px);
}
.modal-enter-active {
  transition: all 0.3s ease;
}
.modal-leave-to {
  opacity: 0;
  transform: translateY(-60px);
}
.modal-leave-active {
  transition: all 0.3s ease;
}
.remove-btn{
  margin-left: 10px;
}

.dustbin{
   position: fixed;
  right: 500px;
  top: 200px;
}

.animate-wrap .animate {
  position: fixed;
  right: 500px;
  top: 200px;
  z-index: 100;
  transition: all 1.5s linear;
}
</style>
