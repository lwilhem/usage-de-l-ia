<script setup lang="ts">
import MistralClient from "@mistralai/mistralai";
import { ref } from "vue";

const activity_name = ref<string>("");
const number_of_questions = ref<number>(0);
const target = ref<string>("");
const objectives = ref<string>("");
const activity = ref<string>("");

const client = new MistralClient(import.meta.env.VITE_PUBLIC_MISTRAL_API_KEY);
const mistral_response = ref<string | undefined>();

const HandleQuestionGeneration = (e: MouseEvent) => {
	console.log(e.target);

	const system_prompt_build = `You are a generator of non-biased questions for a survey to help accomplish the following objectives: ${objectives}.
  You generate the survey for '${activity_name}' which is an entity that does ${activity}.
  You are forbidden from using placeholders in your answers (ex: [Entity Name])
  You NEED to focus on the activity given, which is : ${activity}`;
	const prompt_build = `
    Generate a series of ${number_of_questions} and only ${number_of_questions} questions that targets ${target}`;

	client
		.chat({
			model: "mistral-tiny",
			temperature: 0.2,
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
	<main
		class="w-full h-screen flex items-center justify-evenly bg-zinc-900 text-slate-100"
	>
		<section
			class="flex flex-col items-center justify-center bg-red-500 w-[44rem] h-96 text-zinc-900"
		>
			<label class="flex flex-col">
				<span class="text-sm font-light lowercase font-mono"
					>nom de votre activité</span
				>
				<input type="text" v-model="activity_name" />
			</label>

			<label class="flex flex-col">
				<span class="text-sm font-light lowercase font-mono"
					>nombre de questions</span
				>
				<input type="number" v-model="number_of_questions" />
			</label>

			<label class="flex flex-col">
				<span class="text-sm font-light lowercase font-mono"
					>Cible de vos questions</span
				>
				<input type="text" v-model="target" />
			</label>

			<label class="flex flex-col">
				<span class="text-sm font-light lowercase font-mono"
					>Objectifs / thème</span
				>
				<textarea v-model="objectives" />
			</label>

			<label class="flex flex-col">
				<span class="text-sm font-light lowercase font-mono">Activité</span>
				<input type="text" v-model="activity" />
			</label>

			<button @click="HandleQuestionGeneration">submit form</button>
		</section>

		<section v-if="mistral_response">
			<p>{{ mistral_response }}</p>
		</section>
	</main>
</template>
