<template>
  <div style="width:400px;">
    <el-card class="box-card">
    <template #header>
      <div class="card-header">
        <span class="card-time">{{nowTime}}</span>
        <el-button class="button"  @click="handlerAddTask" text>添加待办事项</el-button>
      </div>
    </template>
    <!-- todo list -->
    <div v-if="tasks.length !== 0">
        <!-- <draggable
          v-model="tasks.value"
          :disabled="draggableDisabled"
          animation="200"
          @start="onStart"
          @end="onEnd"
          @click.stop
        > -->
          <transition-group name="fade" tag="div">
            <el-row
              :class="{ 'clear-transition': !isDrag }"
              v-for="item in tasks"
              :key="item.id"
            >
              <el-col>
                <el-checkbox
                  class="checkbox"
                  v-model="item.checked"
                  @change="handlerCheckboxChange(item)"
                ></el-checkbox>
                <label
                  v-if="!item.isEdit"
                  class="checkbox-text"
                  :class="{ select: item.checked }"
                  @dblclick.stop="handlerEditTask(item)"
                >
                  {{ item.label }}
                </label>
                <el-input
                  v-else
                  v-focus
                  class="text-input"
                  v-model="item.label"
                  placeholder="输入待办任务～"
                  size="mini"
                  @blur="handlerBlur(item)"
                ></el-input>
                <el-icon class="el-icon-delete show-remove-icon" @click="handlerRemoveTask(item)"><Delete /></el-icon>
              </el-col>
            </el-row>
          </transition-group>
        <!-- </draggable> -->
      </div>
      <!-- empty -->
      <el-empty
        v-else
        :image-size="70"
        description="暂无任务列表，快去创建吧～"
      ></el-empty>
  </el-card>
  </div>
</template>
<script setup>
import {computed,nextTick,ref,onBeforeUnmount} from "vue"
import dayjs from "dayjs";
import { v4 as uuidv4 } from "uuid";
import {
  setTasksListLocalstory,
  getTasksListLocalstory,
  updateTasksListLocalstory,
  clearTasksLocalstory,
  setBadgeText,
} from "./utils/index";
import {
  Delete
} from '@element-plus/icons-vue'
const vFocus = {
  mounted: (el) => {
    el.querySelector("input").focus();
  }
}

const nowTime = computed(()=>{
  return dayjs().format("YYYY-MM-DD");
})

const drag = ref(false) 
const isDrag = ref(false)
const tasks = ref([])
const draggableDisabled = ref(false)

const handlerAddTask = () => {
  isDrag.value = true;
  if (tasks.value.length > 29) {
    alert("最多创建 30 个任务哦～");
    return;
  }
  draggableDisabled.value = true;
  tasks.value.unshift({
    id: uuidv4(),
    label: "",
    isEdit: true,
    checked: false,
  });
  console.log(tasks.value);
}
// Start Drag
const onStart = ()=> {
  drag.value = true;
  isDrag.value = true;
}
// End Drag
const onEnd = ()=> {
  drag.value = false;
  isDrag.value = false;
  updateTasksListLocalstory(tasks.value);
}
const handlerCheckboxChange=(item)=> {
  if (item.checked) {
    isDrag.value = true;
    let temp = [];
    const preIndex = tasks.value.findIndex((el) => el.id === item.id);
    if (preIndex > 0) {
      const pre = tasks.value.slice(0, preIndex);
      temp = pre;
    }
    const nextArr = tasks.value.slice(preIndex + 1, tasks.value.length);
    if (nextArr.length > 0) {
      temp.push(...nextArr);
    }
    temp.push(item);
    tasks.value = temp;
  }
  updateTasksListLocalstory(tasks.value);
}
const handlerEditTask=(item)=> {
  if (!item.isEdit) {
    item.isEdit = true;
  }
}
const handlerBlur=(item)=> {
  if (item.label.trim() !== "") {
    item.isEdit = false;
    setTasksListLocalstory(tasks.value);
  } else {
    handlerRemoveTask(item);
  }
  draggableDisabled.value = false;
}
const handlerRemoveTask=(item)=> {
  isDrag.value = true;
  // nextTick(() => {
    const index = tasks.value.findIndex((el) => el.id === item.id);
    tasks.value.splice(index, 1);
    updateTasksListLocalstory(tasks.value);
  // });
}
const init = ()=>{
  getTasksListLocalstory(tasks);
}
init()

onBeforeUnmount(()=>{
  setTasksListLocalstory(tasks.value);
}) 
</script>

<style scoped>
  @import "./common.css";
  /* 动画 */
  .fade-move,
  .fade-enter-active,
  .fade-leave-active {
    transition: all 0.5s cubic-bezier(0.55, 0, 0.1, 1);
  }
  
  .clear-transition {
    transition: unset !important;
  }
  
  .fade-enter {
    opacity: 0;
  }
  .fade-leave-to {
    opacity: 0;
    transform: translateX(-10px);
  }
  .fade-leave-active {
    position: absolute;
  }
  .container {
    width: 350px;
    padding: 0 10px;
    background: #f2f6fc;
    margin: 20px 0 10px 0;
    font-family: "微软雅黑";
  }
  .header-title {
    width: 350px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    font-size: 18px;
    padding: 0 0 10px 10px;
    margin-bottom: 15px;
    color: #2a3a4a;
    font-weight: 500;
    border-bottom: 1px solid #e4e7ed;
    user-select: none;
  }
  .todo-list {
    border-radius: 6px;
    background: #fff;
    padding: 10px;
    height: 300px;
    overflow: auto;
  }
  /* Row */
  .el-row {
    height: 35px;
    display: flex;
    align-items: center;
    border-radius: 4px;
    padding: 0 10px;
    margin-bottom: 5px;
    border-bottom: 1px solid #e4e7ed;
  }
  /* Col */
  .el-col {
    position: relative;
    overflow: hidden;
    display: flex;
    align-items: center;
    padding-right: 20px;
  }
  /* checkbox 文本 */
  .checkbox-text {
    width: 100%;
    color: #606266;
    font-size: 14px;
    font-weight: 500;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
    cursor: move;
    user-select: none;
  }
  /*  移除 icon */
  .el-icon-delete {
    display: none;
    position: absolute;
    right: 0;
    cursor: pointer;
    font-size: 14px;
    color: #ccc;
  }
  /* 移除 icon hover */
  .el-col:hover .show-remove-icon {
    display: block;
  }
  .select {
    text-decoration: line-through;
    color: #ccc;
  }
  .card-header {
    display: flex;
    justify-content: space-between;
  }
  .card-time {
    display: flex;
    align-items: center;
    font-size: 15px;
    font-weight: 500;
    cursor: pointer;
    user-select: none;
  }
  .icon-mdatepicker {
    margin-right: 5px;
  }
  .card-add-icon {
    font-size: 15px;
    font-weight: 500;
  }
  footer {
    margin-top: 10px;
    display: flex;
    font-size: 15px;
    justify-content: space-between;
    color: #afacac;
  }
  
  .icon-github {
    font-size: 16px;
  }
  .icon-dashang1 {
    font-size: 14px;
  }
  .icon-weixin1 {
    font-size: 18px;
  }
  .icon-qingkong1 {
    font-size: 13px;
  }
  
  .reward:hover .icon-dashang1 {
    -webkit-transform: rotateY(360deg);
    transform: rotateY(360deg);
    -webkit-transition: -webkit-transform 0.5s;
    transition: transform 0.5s;
  }
  .clear:hover .icon-qingkong1 {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
    -webkit-transition: -webkit-transform 0.5s;
    transition: transform 0.5s;
  }
</style>