<template>
  <div class="container">
    <el-card>
      <div ref="imageContainer" class="image-wrapper">
        <el-image
            :src="imageUrl"
            class="image"
            @mousedown="startSelection"
            @dragstart.prevent
        />

        <!-- Selection Box -->
        <div
            v-if="isSelecting"
            class="selection-box"
            :style="selectionStyle"
        ></div>

        <!-- Saved Comments with Boxes -->
        <div
            v-for="(comment, index) in comments"
            :key="index"
            class="comment-box"
            :class="{ highlighted: selectedComment === index }"
            :style="boxStyle(comment)"
            @click="selectedComment = index"
        >
          <el-tooltip effect="dark" :content="comment.text" placement="top">
            <el-icon class="comment-icon"><ChatDotRound /></el-icon>
          </el-tooltip>
        </div>

        <!-- Textarea Input -->
        <el-input
            v-if="showInput"
            ref="inputRef"
            v-model="inputText"
            type="textarea"
            class="floating-input"
            :style="inputStyle"
            rows="3"
            @blur="saveComment"
            @keydown.enter.prevent="saveComment"
            placeholder="Enter a comment..."
        />
      </div>
    </el-card>

    <!-- Comments List -->
    <el-card class="comment-list" shadow="hover">
      <h3>Comments</h3>
      <el-empty v-if="comments.length === 0" description="No comments yet" />
      <el-scrollbar v-else>
        <div v-for="(comment, index) in comments" :key="index"
             @click="selectedComment = index"
             class="comment-item"
             :class="{ active: selectedComment === index }">
          <el-text># {{ index + 1 }}: {{ comment.text }}</el-text>
        </div>
      </el-scrollbar>
    </el-card>
  </div>
</template>

<script setup>
import { ref, computed, nextTick, onMounted } from "vue";
import { ChatDotRound } from "@element-plus/icons-vue";

const imageUrl = ref("https://fastly.picsum.photos/id/193/3578/2451.jpg?hmac=M5yoazhwdwMa_27rC5-S50SNFvCy4Kni0wXoa6iVF0g");
const imageContainer = ref(null);
const isSelecting = ref(false);
const selection = ref({ x: 0, y: 0, width: 0, height: 0 });
const startPos = ref({ x: 0, y: 0 });
const showInput = ref(false);
const inputText = ref("");
const comments = ref([]);
const selectedComment = ref(null);
const inputRef = ref(null);

const selectionStyle = computed(() => ({
  top: `${selection.value.y}px`,
  left: `${selection.value.x}px`,
  width: `${selection.value.width}px`,
  height: `${selection.value.height}px`,
}));

const inputStyle = computed(() => ({
  top: `${selection.value.y + selection.value.height}px`,
  left: `${selection.value.x}px`,
}));

const boxStyle = (comment) => ({
  top: `${comment.y}px`,
  left: `${comment.x}px`,
  width: `${comment.width}px`,
  height: `${comment.height}px`,
});

const loadComments = () => {
  const savedComments = localStorage.getItem("imageComments");
  if (savedComments) {
    comments.value = JSON.parse(savedComments);
  }
};

const saveCommentsToLocalStorage = () => {
  localStorage.setItem("imageComments", JSON.stringify(comments.value));
};

const startSelection = (event) => {
  event.preventDefault();
  isSelecting.value = true;
  const rect = imageContainer.value.getBoundingClientRect();
  startPos.value = { x: event.clientX - rect.left, y: event.clientY - rect.top };
  selection.value = { x: startPos.value.x, y: startPos.value.y, width: 0, height: 0 };

  document.addEventListener("mousemove", updateSelection);
  document.addEventListener("mouseup", endSelection);
};

const updateSelection = (event) => {
  if (!isSelecting.value) return;
  const rect = imageContainer.value.getBoundingClientRect();
  const currentX = event.clientX - rect.left;
  const currentY = event.clientY - rect.top;

  selection.value.width = Math.abs(currentX - startPos.value.x);
  selection.value.height = Math.abs(currentY - startPos.value.y);
  selection.value.x = Math.min(currentX, startPos.value.x);
  selection.value.y = Math.min(currentY, startPos.value.y);
};

const endSelection = () => {
  isSelecting.value = false;
  showInput.value = true;

  document.removeEventListener("mousemove", updateSelection);
  document.removeEventListener("mouseup", endSelection);

  nextTick(() => {
    inputRef.value?.focus();
  });
};

const saveComment = () => {
  if (inputText.value.trim()) {
    comments.value.push({ ...selection.value, text: inputText.value.trim() });
    saveCommentsToLocalStorage();
  }
  showInput.value = false;
  inputText.value = "";
};

onMounted(loadComments);
</script>

<style scoped>
.container {
  display: flex;
  gap: 20px;
  justify-content: center;
}

.image-wrapper {
  position: relative;
  display: inline-block;
}

.image {
  width: 500px;
  height: auto;
  cursor: crosshair;
  user-select: none;
}

.selection-box {
  position: absolute;
  border: 2px dashed blue;
  background-color: rgba(0, 0, 255, 0.1);
  pointer-events: none;
}

.comment-box {
  position: absolute;
  border: 2px solid red;
  background-color: rgba(255, 0, 0, 0.1);
  display: flex;
  justify-content: center;
  align-items: center;
}

.comment-icon {
  color: red;
  font-size: 20px;
  cursor: pointer;
}

.floating-input {
  position: absolute;
  width: 150px;
  z-index: 10;
}

.comment-list {
  width: 300px;
  padding: 10px;
}

.comment-item {
  padding: 10px;
  border-bottom: 1px solid #ddd;
  cursor: pointer;
  transition: background 0.3s;
}

.comment-item:hover {
  background: #f5f5f5;
}

.active {
  background-color: yellow !important;
}

.highlighted {
  border: 3px solid yellow !important;
  background-color: rgba(255, 255, 0, 0.3) !important;
  box-shadow: 0 0 10px rgba(255, 255, 0, 0.8);
  transition: all 0.3s ease-in-out;
}
</style>
