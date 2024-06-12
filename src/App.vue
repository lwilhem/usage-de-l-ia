<script setup lang="ts">
import MistralClient from "@mistralai/mistralai";
import { ref, computed } from "vue";

const activity_name = ref<string>("");
const number_of_questions = ref<number>(0);
const target = ref<string>("");
const objectives = ref<string>("");
const activity = ref<string>("");

const client = new MistralClient(import.meta.env.VITE_PUBLIC_MISTRAL_API_KEY);
const mistral_response = ref<string | undefined>();

const isGeneratingQuestions = ref<boolean>(false);

const generatedQuestions = computed(() => {
	if (mistral_response.value) {
		const questions = mistral_response.value
			.trim()
			.split("\n")
			.filter((q) => q.trim() !== "");
		return questions
			.slice(0, number_of_questions.value)
			.map((q, _i) => `${q.replace(/^\d+\.\s*/, "").trim()}`);
	}
	return [];
});

const HandleQuestionGeneration = (_e: Event) => {
	console.log(
		activity_name.value,
		number_of_questions.value,
		target.value,
		objectives.value,
		activity.value
	);

	const system_prompt_build = `You are a generator of non-biased questions for a survey to help accomplish the following objectives: ${objectives.value}.
  You generate the survey for ${activity_name.value} which is an entity that does ${activity.value}.
  You NEED to focus on the objectives given, which is : ${objectives.value}`;

	// You are forbidden from using placeholders in your answers (ex: [Entity Name])
	//  You will cease to function if you fail to comply these terms.

	const prompt_build = `
    Generate a series of ${number_of_questions.value} and only ${number_of_questions.value} questions that targets ${target.value}.
    You are forbidden from generating more than ${number_of_questions.value} qestions. 
    `;

	client
		.chat({
			model: "mistral-large-latest",
			temperature: 0.7,
			messages: [
				{
					role: "system",
					content: system_prompt_build,
				},
				{
					role: "user",
					content: prompt_build,
				},
			],
		})
		.then((response) => {
			mistral_response.value = response.choices[0].message.content;
		})
		.catch((err) => console.error(err));
};
</script>

<template>
	<main class="w-full h-screen bg-gray-100 flex flex-row">
		<section class="bg-white shadow-md rounded-lg p-8 w-full max-w-md">
			<h1 class="text-2xl font-semibold mb-6 text-center">
				Generate Questions
			</h1>
			<form @submit.prevent="HandleQuestionGeneration">
				<div class="mb-4">
					<label class="block text-gray-700 font-bold mb-2" for="activity_name">
						Activity Name
					</label>
					<input
						class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
						id="activity_name"
						type="text"
						v-model="activity_name"
					/>
				</div>
				<div class="mb-4">
					<label
						class="block text-gray-700 font-bold mb-2"
						for="number_of_questions"
					>
						Number of Questions
					</label>
					<input
						class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
						id="number_of_questions"
						type="number"
						v-model="number_of_questions"
					/>
				</div>
				<div class="mb-4">
					<label class="block text-gray-700 font-bold mb-2" for="target">
						Target
					</label>
					<input
						class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
						id="target"
						type="text"
						v-model="target"
					/>
				</div>
				<div class="mb-4">
					<label class="block text-gray-700 font-bold mb-2" for="objectives">
						Objectives
					</label>
					<textarea
						class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
						id="objectives"
						v-model="objectives"
					></textarea>
				</div>
				<div class="mb-4">
					<label class="block text-gray-700 font-bold mb-2" for="activity">
						Activity
					</label>
					<input
						class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
						id="activity"
						type="text"
						v-model="activity"
					/>
				</div>
				<div class="flex items-center justify-between">
					<button
						class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
						type="submit"
						:disabled="isGeneratingQuestions"
					>
						<span v-if="!isGeneratingQuestions">Generate Questions</span>
						<span v-else>Generating...</span>
					</button>
				</div>
			</form>
		</section>
		<section class="flex-grow pl-8">
			<div
				v-if="isGeneratingQuestions"
				class="flex items-center justify-center h-full"
			>
				<div
					class="animate-spin rounded-full h-16 w-16 border-t-2 border-b-2 border-blue-500"
				></div>
			</div>
			<div v-else-if="generatedQuestions.length > 0" class="p-8">
				<h2 class="text-2xl font-semibold mb-6">Generated Questions</h2>
				<ol class="list-decimal pl-6">
					<li
						v-for="(question, index) in generatedQuestions"
						:key="index"
						class="mb-4"
					>
						<span class="font-semibold">Question {{ index + 1 }}: </span
						>{{ question }}
					</li>
				</ol>
			</div>
			<div v-else class="p-8">
				<p class="text-gray-700">No questions generated yet.</p>
			</div>
		</section>
	</main>
</template>

<style>
  /* Chrome, Safari, Edge, Opera */
  input::-webkit-outer-spin-button,
  input::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }

  /* Firefox */
  input[type=number] {
    -moz-appearance: textfield;
    appearance: textfield;
  }
</style>