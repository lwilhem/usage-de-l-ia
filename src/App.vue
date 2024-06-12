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

const HandleQuestionGeneration = (_e: MouseEvent) => {
	console.log(activity_name.value, number_of_questions.value, target.value, objectives.value, activity.value);

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
			mistral_response.value = response.choices[0].message.content
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
