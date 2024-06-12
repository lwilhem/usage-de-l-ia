<script setup lang="ts">
import MistralClient, { Message } from '@mistralai/mistralai';
import { ref } from 'vue';

const mistral_prompt = ref("")

const mistral = new MistralClient(import.meta.env.VITE_PUBLIC_MISTRAL_API_KEY)
const model = 'open-mistral-7b';
const messages: Message[] = [
  {
    role: 'system',
    content:
      "You're a sport expert with more than 40 years experience, and give advice and programm for every person depand on their size, age, and weight.",
  },
  {
    role: 'user',
    content:
      "I'm looking for a workout program that will help me lose weight and build muscle. I'm 22 years old, 1m84, and weigh 85kg. Can you help me?",
  },
];

const test = async () => {

  const chatResponse =  mistral.chatStream({
    model: model,
    messages: messages,
    maxTokens: 20,
  });

  for await (const chunk of chatResponse) {
    if (chunk.choices[0].delta.content !== undefined) {
      const streamText = chunk.choices[0].delta.content;
      console.log(streamText)
    }
  }

  return chatResponse;
}

</script>

<template>
  <main class="flex items-center justify-center w-full h-screen bg-zinc-900 text-slate-100">
    <section class="flex items-center justify-evenly">
      <input type="text" v-model="mistral_prompt" class="mx-4">
      <button class="mx-4" @click="() => {
        console.log(mistral_prompt)
      }">Submit Prompt</button>
    </section> 
  </main>
</template>