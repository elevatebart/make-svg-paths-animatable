<script setup lang="ts">
import { computed, ref } from "vue";
import * as Diff from "diff";
import PathMorphers from "./PathMorphers.vue";

const pathA = ref(
  "M17.9146 19.9573L14 18H13C11.3431 18 10 16.6569 10 15V14H14C15.6569 14 17 12.6569 17 11V8H19C20.6569 8 22 9.34315 22 11V15C22 16.6569 20.6569 18 19 18V19.2865C19 19.844 18.4133 20.2066 17.9146 19.9573Z"
);
const pathB = ref(
  "M18.8641 21.3185L15 19H13C11.3431 19 10 17.6569 10 16V15H13C15.7614 15 18 12.7614 18 10V6H21C22.6569 6 24 7.34315 24 9V16C24 17.6569 22.6569 19 21 19H20V20.6754C20 21.2583 19.364 21.6184 18.8641 21.3185Z"
);

function makePathArray(pathInput: string) {
  return pathInput.replace(/([a-zA-Z])/g, "\n$1").split("\n");
}

function removeNumbers(pathInput: string[]) {
  return pathInput.map((line) => line[0]).join("\n");
}

const arrayA = computed(() => {
  return makePathArray(pathA.value);
});

const arrayB = computed(() => {
  return makePathArray(pathB.value);
});

const diff = computed(() => {
  return Diff.diffChars(
    removeNumbers(arrayA.value),
    removeNumbers(arrayB.value)
  );
});

function makeEmptyPath(value: string) {
  if (value.trim() === "") {
    return value;
  }
  const letter = value[0].toLowerCase();
  return letter === "h" || letter === "v"
    ? `${letter}0`
    : letter === "l" || letter === "m"
    ? `${letter}0 0`
    : letter === "c"
    ? `${letter}0 0 0 0 0 0`
    : letter === "a"
    ? `${letter}0 0 0 0 0 0 0 0 0 0`
    : value;
}

const fixedPathA = computed(() => {
  let offset = -1;
  return diff.value
    .map((part) => {
      if (part.added) {
        return part.value
          .split("\n")
          .map((line) => {
            return makeEmptyPath(line);
          })
          .join("\n");
      }
      return part.value
        .split("\n")
        .map(() => {
          return arrayA.value[offset++];
        })
        .join("\n");
    })
    .join("\n");
});

const fixedPathB = computed(() => {
  let offset = 0;
  return diff.value
    .map((part) => {
      if (part.removed) {
        return part.value
          .split("\n")
          .map((line) => {
            return makeEmptyPath(line);
          })
          .join("\n");
      }
      return part.value
        .split("\n")
        .map(() => {
          return arrayB.value[offset++];
        })
        .join("\n");
    })
    .join("\n");
});

const demoState = ref<"A" | "B">("A");
</script>

<template>
  <div class="inputs">
    <textarea v-model="pathA" />
    <textarea v-model="pathB" />
    <pre>{{ pathA.replace(/([a-zA-Z])/g, "\n$1") }}</pre>
    <pre>{{ pathB.replace(/([a-zA-Z])/g, "\n$1") }}</pre>
    <svg
      fill-rule="evenodd"
      height="100"
      width="100"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
      fill="red"
    >
      <path :d="pathA" />
    </svg>
    <svg
      fill-rule="evenodd"
      height="100"
      width="100"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
      fill="green"
    >
      <path :d="pathB" />
    </svg>
    <pre>
        <span v-for="line of fixedPathA.split('\n').filter(s => s.trim().length > 0)" class="line">{{ line }}</span>
      </pre>
    <pre>
        <span v-for="line of fixedPathB.split('\n').filter(s => s.trim().length > 0)" class="line">{{ line }}</span>
      </pre>
    <svg
      fill-rule="evenodd"
      height="100"
      width="100"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
      fill="red"
    >
      <path :d="fixedPathA" />
    </svg>

    <svg
      fill-rule="evenodd"
      height="100"
      width="100"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
      fill="green"
    >
      <path :d="fixedPathB" />
    </svg>
  </div>
  <button
    @click="
      () => {
        demoState = demoState === 'A' ? 'B' : 'A';
      }
    "
  >
    <svg
      fill-rule="evenodd"
      height="100"
      width="100"
      viewBox="0 0 24 24"
      xmlns="http://www.w3.org/2000/svg"
      fill="indigo"
    >
      <PathMorphers
        :d="fixedPathA"
        :dAnimated="fixedPathB"
        :animated="demoState === 'B'"
      />
    </svg>
  </button>
</template>

<style scoped>
.inputs {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  justify-items: center;
  gap: 40px;
}

.inputs textarea {
  width: 400px;
  height: 200px;
}

pre {
  text-align: left;
  white-space: pre-wrap;
  counter-reset: linecounter;
}

pre > span.line {
  display: block;
}

pre > span.line:before {
  display: inline-block;
  width: 30px;
  content: counter(linecounter);
  counter-increment: linecounter;
  text-align: right;
  padding-right: 1rem;
}
</style>
