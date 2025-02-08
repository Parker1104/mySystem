<script setup lang="ts">
import OpenAI from "openai";
import { onMounted, ref } from "vue";
import VueMarkdownIt from "vue3-markdown-it";
import { NButton, NInput } from "naive-ui";
const chatMessage = ref<any[]>([]);
const message = ref<string>("");
const loading = ref<boolean>(false);
const doQuery = async () => {
  const openai = new OpenAI({
    baseURL: "https://api.deepseek.com",
    // 个人测试
    apiKey: "",
    dangerouslyAllowBrowser: true,
  });
  chatMessage.value.push({type: 'user', message: message.value});
  loading.value = true;
  const response = await openai.chat.completions.create({
    messages: [{ role: "user", content: message.value }],
    stream: true,
    temperature: 1.3,
    model: "deepseek-chat",
  });
  chatMessage.value.push({type: 'system', message: ''});

  for await (const chunk of response) {
    chatMessage.value[chatMessage.value.length - 1].message +=
      chunk.choices[0]?.delta?.content || "";
    if (chunk.choices[0].finish_reason === "stop") {
      message.value = "";
      loading.value = false;
    }
  }
};
</script>
<template>
  <div class="page">
    <div class="contianer">
      <div class="top">
        <div v-for="i in chatMessage" :class="i.type === 'user' ? 'right' : 'left'" >
          <div v-if="i.type === 'user'">
            {{ i.message }}
          </div>
          <VueMarkdownIt v-else :source="i.message" />
        </div>
      </div>
      <div class="bottom">
        <NInput
          v-model:value="message"
          type="textarea"
          placeholder="向deepseek发送消息"
          @keydown.enter="doQuery"
          :loading="loading"
          :disabled="loading"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.page {
  width: 100%;
  display: flex;
  justify-content: center;
}
.contianer {
  width: 600px;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.top {
  flex: 1;
  overflow: auto;
  display: flex;
  flex-direction: column;
  padding: 12px 0;
}
.bottom {
  padding: 8px;
}
.left {
  align-self: flex-start;
}
.right {
  align-self: flex-end;
  background-color: rgba(24, 160, 88, 0.15);
  padding: 8px 12px;
  border-radius: 8px;
}
::-webkit-scrollbar {
  width: 10px !important;
  height: 10px !important;
}
</style>
