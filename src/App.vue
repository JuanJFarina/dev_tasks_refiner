<script setup lang="ts">
import { ref } from 'vue';
import { GoogleGenerativeAI } from "@google/generative-ai"

const genAI = new GoogleGenerativeAI(import.meta.env.VITE_API_KEY);
const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

// Reactive variables for form data and API response
const title = ref('');
const description = ref('');
const acceptanceCriteria = ref('');
const showConfig = ref(false);
const refinementResult = ref('');
const isLoading = ref(false);
const errorMessage = ref('');

// Toggle configuration visibility
const toggleConfig = () => {
  showConfig.value = !showConfig.value;
};

// Construct the prompt
const constructPrompt = () => `
You are a Scrum Master with huge expertise in different software engineering disciplines. You are given this task:
  
Title: ${title.value}
Description: ${description.value}
Acceptance Criteria: ${acceptanceCriteria.value}

Revise the next points:

- Check that the title and description match each other, otherwise adjust the title.
- Check that the description explicitly states which situation must be solved and/or what actions must be taken, considering resources and/or on which repository or place work.
- Check that the acceptance criteria details exactly what the developer's solution should address and what QA should test, specifying the tools for QA testing if applicable.

DO NOT add details that weren't in the original task. If some detail is missing, just put "Needs Clarification".

Your response should be in the following JSON format:

{
  "title": "title of the task",
  "description": {
    "situation_to_solve": "the situation that gives birth to the task",
    "actions_to_be_done": "the actions that must be addressed by the developer",
    "resources_and_repos": "resources and repositories that the developer must work with"
  },
  "acceptance criteria": {
    "dev_ac": "criteria to consider done the developer tasks",
    "qa_ac": "criteria to consider done the QA tasks"
  }
}`;

// API call to refine the task
const refineTask = async () => {
  isLoading.value = true;
  errorMessage.value = '';
  
  try {
    const prompt = constructPrompt();
    const result = await model.generateContent(prompt);
    refinementResult.value = result.response.text();
  } catch (error) {
    errorMessage.value = 'Failed to refine the task. Please try again.';
    console.error(error);
  } finally {
    isLoading.value = false;
  }
};

// Reset form
const resetForm = () => {
  title.value = '';
  description.value = '';
  acceptanceCriteria.value = '';
  refinementResult.value = '';
};
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
    <div class="wrapper">
      <h1>Create Developer Task</h1>
    </div>
  </header>

  <main>
    <div class="task-form">
      <label for="title">Title:</label>
      <input id="title" v-model="title" placeholder="Enter task title" />

      <label for="description">Description:</label>
      <textarea id="description" v-model="description" placeholder="Enter task description"></textarea>

      <label for="acceptance-criteria">Acceptance Criteria:</label>
      <textarea id="acceptance-criteria" v-model="acceptanceCriteria" placeholder="Enter acceptance criteria"></textarea>

      <button @click="toggleConfig">
        {{ showConfig ? 'Hide' : 'Show' }} Configuration
      </button>

      <div v-if="showConfig" class="config-items">
        <!-- Add configuration options here -->
        <p>Configuration Items (TBD)</p>
      </div>

      <button @click="refineTask" :disabled="isLoading">
        {{ isLoading ? 'Refining...' : 'Refine!' }}
      </button>

      <button @click="resetForm" :disabled="isLoading">Reset</button>

      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
    </div>

    <div class="result-section" v-if="refinementResult">
      <h2>Refinement Result:</h2>
      <pre>{{ refinementResult }}</pre>
    </div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
  text-align: center;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

.task-form {
  max-width: 600px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
}

.task-form label {
  margin: 0.5rem 0 0.2rem;
  font-weight: bold;
}

.task-form input, .task-form textarea {
  padding: 0.5rem;
  margin-bottom: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: 100%;
}

button {
  margin: 0.5rem 0;
  padding: 0.5rem;
  background-color: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button[disabled] {
  background-color: #a9a9a9;
  cursor: not-allowed;
}

button:hover:enabled {
  background-color: #368f6e;
}

.config-items {
  margin-top: 1rem;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f9f9f9;
}

.result-section {
  margin-top: 2rem;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #e6f7ff;
}

pre {
  white-space: pre-wrap;
}

.error {
  color: red;
  margin-top: 1rem;
}
</style>
