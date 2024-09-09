<template>
  <TheWarning :opacityWarning="opacityW" :translateWarning="translateW" />
  <div class="container">
    <div class="center-logo" :style="{ opacity: opacitybg }">
      <img src="/logo.png" class="gygole" />
      I am G-GPT!
    </div>

    <div class="answers-column">
      <div
        v-for="(answer, index) in modifiedAnswers"
        :key="index"
        :class="classanswer"
        :style="{
          display: dispanswer,
          opacity: answer.opacity,
          transform: answer.translate,
        }"
      >
        <template v-if="answer.codeBlock !== ''">
          <div v-if="answer.beforeBackticks">{{ answer.beforeBackticks }}</div>
          <pre>
            <code class="text-code">{{ answer.codeBlock }}</code>
            <button 
              class="copy-code" 
              @mousedown="startHolding" 
              @mouseup="stopHolding" 
              @mouseleave="stopHolding"
              @click="copyText(answer.codeBlock)"
            >
              <svg class="ico-copy" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">
                <path :fill="colorCopy" d="M384 336l-192 0c-8.8 0-16-7.2-16-16l0-256c0-8.8 7.2-16 16-16l140.1 0L400 115.9 400 320c0 8.8-7.2 16-16 16zM192 384l192 0c35.3 0 64-28.7 64-64l0-204.1c0-12.7-5.1-24.9-14.1-33.9L366.1 14.1c-9-9-21.2-14.1-33.9-14.1L192 0c-35.3 0-64 28.7-64 64l0 256c0 35.3 28.7 64 64 64zM64 128c-35.3 0-64 28.7-64 64L0 448c0 35.3 28.7 64 64 64l192 0c35.3 0 64-28.7 64-64l0-32-48 0 0 32c0 8.8-7.2 16-16 16L64 464c-8.8 0-16-7.2-16-16l0-256c0-8.8 7.2-16 16-16l32 0 0-48-32 0z"/>
              </svg>
            </button>
          </pre>
          <div v-if="answer.afterBackticks">{{ answer.afterBackticks }}</div>
        </template>
        <template v-else>
          <span class="text">{{ answer.text }}</span>
        </template>
      </div>
    </div>

    <div class="chat">
      <div class="input-message">
        <i class="icon">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
            <path
              fill="#ffffff"
              d="M64 0C28.7 0 0 28.7 0 64L0 352c0 35.3 28.7 64 64 64l96 0 0 80c0 6.1 3.4 11.6 8.8 14.3s11.9 2.1 16.8-1.5L309.3 416 448 416c35.3 0 64-28.7 64-64l0-288c0-35.3-28.7-64-64-64L64 0z"
            />
          </svg>
        </i>
        <div class="cont-inputs">
          <div
            class="cont-input"
            id="cont-input"
            :style="{ height: heightInput + 'px' }"
          >
            <input
              v-for="(input, index) in inputs"
              :key="index"
              type="text"
              :id="`code-${index + 1}`"
              name="code[]"
              class="input-text"
              v-model="inputs[index]"
            />
          </div>
          <div class="bottom-cont" :style="{ display: dispBottomcont }"></div>
        </div>
        <button class="send-btn icon" @click="sendAnswer">
          <i class="icon2">
            <svg
              v-if="isSenttext"
              class="loading"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 448 512"
            >
              <path
                fill="#ffffff"
                d="M8 256a56 56 0 1 1 112 0A56 56 0 1 1 8 256zm160 0a56 56 0 1 1 112 0 56 56 0 1 1 -112 0zm216-56a56 56 0 1 1 0 112 56 56 0 1 1 0-112z"
              />
            </svg>
            <svg
              v-else
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 448 512"
            >
              <path
                fill="#ffffff"
                d="M438.6 278.6c12.5-12.5 12.5-32.8 0-45.3l-160-160c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L338.8 224 32 224c-17.7 0-32 14.3-32 32s14.3 32 32 32l306.7 0L233.4 393.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0l160-160z"
              />
            </svg>
          </i>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, onMounted, onUnmounted, computed } from "vue";
import TheWarning from "./TheWarning.vue";

const opacitybg = ref(1);
const inputs = ref([""]);
const dispanswer = ref("block");
const classanswer = ref("answer");
const answers = ref([]);
const dispBottomcont = ref("none");
const heightInput = ref(20);
const loading = ref(false);
const isSenttext = ref(false);
const opacityW = ref(0.1);
const translateW = ref("0px -55px");
const colorCopy = ref("white");
let intervalId = null;

const copyText = async (text) => {
  try {
    await navigator.clipboard.writeText(text);
    console.log("Text copied to clipboard");
  } catch (err) {
    console.error("Failed to copy text: ", err);
  }
};

const startHolding = () => {
  colorCopy.value = "black";
  intervalId = setInterval(() => {
    console.log("Button is being held down");
  }, 100);
};

const stopHolding = () => {
  colorCopy.value = "white";
  if (intervalId) {
    clearInterval(intervalId);
    intervalId = null;
  }
};

const splitText = (text) => {
  const backtickRegex = /```/g;
  const parts = text.split(backtickRegex);

  if (parts.length === 1) {
    return { beforeBackticks: text, codeBlock: "", afterBackticks: "" };
  }

  const codeBlock = parts[1] ? parts[1].trim() : "";
  const afterBackticks = parts.slice(2).join("").trim();

  return {
    beforeBackticks: parts[0] || "",
    codeBlock,
    afterBackticks,
  };
};

const modifiedAnswers = computed(() =>
  answers.value.map((answer) => ({
    ...answer,
    ...splitText(answer.text),
  }))
);

const addInput = () => {
  inputs.value.push("");
  heightInput.value += 30;
  dispBottomcont.value = "flex";

  nextTick(() => {
    const chatSection = document.querySelector(".chat");
    if (chatSection) {
      chatSection.scrollTo({
        top: chatSection.scrollHeight,
        behavior: "smooth",
      });
    }

    const lastInput = document.querySelector(`#code-${inputs.value.length}`);
    if (lastInput) {
      lastInput.focus();
    }
  });
};

const removeInput = () => {
  if (inputs.value.length > 1) {
    inputs.value.pop();
    heightInput.value -= 30;

    if (inputs.value.length === 1) {
      dispBottomcont.value = "none";
    }

    nextTick(() => {
      const chatSection = document.querySelector(".chat");
      if (chatSection) {
        chatSection.scrollTo({
          top: chatSection.scrollHeight,
          behavior: "smooth",
        });
      }

      const lastInput = document.querySelector(`#code-${inputs.value.length}`);
      if (lastInput) {
        lastInput.focus();
      }
    });
  }
};

const handleKeydown = (event) => {
  if (event.key === "Enter") {
    event.preventDefault();
    addInput();
  }
};

const handleKeyup = (event) => {
  if (event.key === "Backspace") {
    removeInput();
  }
};

const scrollToBottom = () => {
  nextTick(() => {
    const answersColumn = document.querySelector(".answers-column");
    if (answersColumn) {
      answersColumn.scrollTop = answersColumn.scrollHeight;
    }
  });
};

const sendAnswer = async () => {
  isSenttext.value = true;

  const combinedText = inputs.value.join(" ").trim();

  if (combinedText === "") {
    isSenttext.value = false;
    opacityW.value = 1;
    translateW.value = "0px 75px";
    setTimeout(() => {
      opacityW.value = 0.1;
      translateW.value = "0px -55px";
    }, 2000);

    return;
  }

  try {
    const response = await fetch("https://xgptback.vercel.app/api/ask/", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ question: combinedText }),
    });

    if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);

    const data = await response.json();

    if (data.answer) {
      const output = data.answer.trim();
      if (output) {
        answers.value.push({
          text: output,
          opacity: 0,
          translate: "0px 10px",
        });

        setTimeout(() => {
          opacitybg.value = 0;
          classanswer.value = "answer active";
          const lastAnswer = answers.value[answers.value.length - 1];
          lastAnswer.opacity = 1;
          lastAnswer.translate = "0px 0px";

          scrollToBottom();
          loading.value = false;
          isSenttext.value = false;
        }, 10);
        inputs.value = [inputs.value[0]];
        heightInput.value = 20;
        dispBottomcont.value = "none";

        const firstInput = document.querySelector(`#code-1`);

        firstInput.value = "";
      }
    }
  } catch (error) {
    console.error("Error making request:", error.message);
    isSenttext.value = false;
  }
};

onMounted(() => {
  const inputContainer = document.querySelector("#cont-input");
  if (inputContainer) {
    inputContainer.addEventListener("keydown", handleKeydown);
    inputContainer.addEventListener("keyup", handleKeyup);
  }
});

onUnmounted(() => {
  const inputContainer = document.querySelector("#cont-input");
  if (inputContainer) {
    inputContainer.removeEventListener("keydown", handleKeydown);
    inputContainer.removeEventListener("keyup", handleKeyup);
  }
});
</script>

<style scoped>
@keyframes load {
  from {
    opacity: 0.1;
  }
  to {
    opacity: 1;
  }
}
.loading {
  opacity: 0.1;
  animation: load 1s infinite alternate ease-in-out;
}
.ico-copy {
  width: 15px;
  height: 15px;
  padding: 7px 10px 10px 7px;
  translate: 0px -31px;
}
.text-code {
  max-width: 400px;
}
.copy-code {
  width: 30px;
  height: 30px;
  margin-left: 10px;
  border-radius: 5px;
  background-color: black;
  color: white;
  border: 1px solid grey;
  transition: all 0.5s ease;
}
.copy-code:active {
  background-color: white;
}
pre {
  background-color: black;
  padding: 10px;
  position: relative;
  margin: 5px;
  display: flex;
  justify-content: space-between;
  border-radius: 4px;
  white-space: pre-wrap;
}
.cont-inputs {
  width: 100%;
}
.answers-column {
  width: 100%;
  height: auto;
  overflow-y: auto;
  padding-top: 60px;
}
.bottom-cont {
  width: 100%;
  height: 5px;
}
.container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  position: relative;
  overflow-y: auto;
}

.chat {
  height: 50px;
  border-top: 1px solid grey;
  background-color: #000;
  position: relative;
}
.center-logo {
  width: 100%;
  display: flex;
  position: absolute;
  top: 160px;
  gap: 15px;
  align-items: center;
  color: white;
  flex-direction: column;
  font-family: "Kanit";
  transition: all 0.5s ease;
}
.gygole {
  width: 50px;
  transition: all 0.5s ease;
}

.input-message {
  width: 100%;
  display: flex;
  gap: 10px;
  background-color: black;
  height: 50px;
}

.cont-input {
  margin-top: 10px;
  width: 100%;

  margin-bottom: 10px;
  padding-bottom: 10px;
  border-radius: 7px;
  background-color: #2e2e2e;
}

.send-btn {
  width: 60px;
  height: 30px;
  margin-top: 10px;
  margin-right: 15px;
  margin-bottom: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 7px;
  border: 0;
  background-color: #2e2e2e;
  transition: all 0.2s ease;
}

.send-btn:active {
  background-color: #4b4b4b;
  scale: 0.9;
}

.answer {
  width: auto;
  max-width: 70%;
  height: auto;
  padding: 10px;
  margin: 10px;
  opacity: 0;
  border-radius: 10px;
  color: white;
  font-family: "Kanit", "Inter";
  font-weight: 300;
  transform: translateY(50px);
  white-space: normal;
  word-wrap: break-word;
  word-break: break-word;
  background-color: #2e2e2e;
  transition: all 0.5s ease;
}

.answer.active {
  opacity: 1;
  transform: translateY(0px);
}

.input-text {
  width: 100%;
  height: 30px;
  border: 0;
  color: white;
  white-space: normal;
  word-wrap: break-word;
  word-break: break-word;
  padding-left: 5px;
  font-family: "Kanit", "Inter";
  background-color: transparent;
  transition: all 0.5s ease;
}
input:focus {
  outline: none;
}

.icon {
  display: flex;
  margin-left: 10px;
  margin-top: 10px;
  width: 60px;
  height: 30px;
}

.icon2 {
  display: flex;
  width: 22.5px;
  height: 22.5px;
}
</style>
