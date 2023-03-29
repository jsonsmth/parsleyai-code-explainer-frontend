<template>
  <div class="code-explainer">
    <div>
    <header class="header-container">
      <img src="/parsley-svgrepo-com.svg" alt="Logo" class="logo" />
      <h1>Parsley Codesplainer</h1>
    </header>
    </div>
    <div class="code-input">
      <label for="code-snippet">Paste your code snippet:</label>
      <textarea maxlength="10000" id="code-snippet" v-model="codeSnippet"></textarea>
    </div>
    <div class="button-container">
<button :disabled="loading" @click="submitCode">
  <div class="button-content">
    <span class="text" v-show="!loading">Explain Code</span>
    <span class="spinner-container" v-show="loading">
      <!-- Inline spinner SVG or use an imported spinner SVG -->
      <svg class="spinner" viewBox="0 0 50 50">
        <circle class="path" cx="25" cy="25" r="20" fill="none" stroke-width="5"></circle>
      </svg>
    </span>
  </div>
</button>
    </div>
    <div class="code-explanation" v-show="true">
      <h2>Explanation:</h2>
      <textarea readonly ref="explanationArea" class="explanation-textarea" v-model="explanation"></textarea>
    </div>
    <div v-if="isNonProduction && rateLimit.remainingRequestsMsg && rateLimit.remainingTokensMsg" class="rate-limit">
      <p>{{ rateLimit.remainingRequestsMsg }}</p>
      <p>{{ rateLimit.remainingTokensMsg }}</p>
    </div>
  </div>
</template>




<script>
import axios from 'axios';

export default {
  data() {
    return {
      codeSnippet: '',
      explanation: '',
      rateLimit: {
        remainingRequestsMsg: '',
        resetRequestsMsg: '',
        remainingTokensMsg: '',
        resetTokensMsg: '',
      },
      loading: false,
    };
  },
  computed: {
    isNonProduction() {
      return process.env.NODE_ENV !== 'production';
    },
  },
  methods: {
    async submitCode() {
        this.loading = true;
        try {
            const backendUrl = process.env.NODE_ENV === 'production'
            ? process.env.VUE_APP_PROD_BACKEND_URL
            : 'http://localhost:3001';
            const response = await axios.post(`${backendUrl}/api/interpret-code`, {
            codeSnippet: this.codeSnippet,
            });
            const rateLimit = response.data.rateLimit;
            if (rateLimit) {
            this.rateLimit.remainingRequestsMsg = rateLimit.remainingRequestsMsg;
            this.rateLimit.resetRequestsMsg = rateLimit.resetRequestsMsg;
            this.rateLimit.remainingTokensMsg = rateLimit.remainingTokensMsg;
            this.rateLimit.resetTokensMsg = rateLimit.resetTokensMsg;
            }
            setTimeout(() => {
            this.explanation = response.data.explanation;
            }, 500); // Delay the display of the explanation
        } catch (error) {
            console.error('Error interpreting code snippet:', error);
        } finally {
            this.loading = false;
        }
    },
  },
};
</script>

<style scoped>

.code-explainer {
  display: flex;
  flex-direction: column;
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.code-explanation {
  visibility: visible;

}

.logo {
  width: 50px;
  height: auto;
}

header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.header-container {
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

.logo {
  margin-right: 1rem;
}

.code-input {
  margin: 1rem 0;
}

textarea,
.explanation-textarea {
  width: 100%;
  min-height: 20vh;
  padding: 1rem;
  font-family: monospace;
  border: 1px solid #ccc;
  resize: vertical;
  box-sizing: border-box; /* Add this property */
}

button {
  width: 100%;
  padding: 0.5rem 1rem;
  background-color: #1e90ff;
  color: #fff;
  border: none;
  cursor: pointer;
  box-sizing: border-box;
  height: 3rem; /* Set a fixed height */
  position: relative; /* Add this property */
}

.button-content {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.text,
.spinner-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}


button:hover {
  background-color: #1c86ee;
}

.button-container {
  width: 100%;
  overflow: hidden; /* Add this property */
  position: relative;
}

.code-explanation {
  margin-top: 2rem;
}

.spinner {
  width: 20px;
  height: 20px;
  animation: spin 1s linear infinite;
}

.spinner .path {
  stroke: #fff;
  stroke-linecap: round;
  animation: dash 1.5s ease-in-out infinite;
}

@keyframes spin {
  100% {
    transform: rotate(360deg);
  }
}

@keyframes dash {
  0% {
    stroke-dasharray: 1, 150;
    stroke-dashoffset: 0;
  }
  50% {
    stroke-dasharray: 90, 150;
    stroke-dashoffset: -35;
  }
  100% {
    stroke-dasharray: 1, 150;
    stroke-dashoffset: -124;
  }
}

.rate-limit {
  margin-top: 2rem;
  border: 1px solid #ccc;
  padding: 1rem;
  display: flex;
  flex-direction: column;
}

.rate-limit p {
  margin: 0;
  padding: 0.5rem 0;
  font-size: 0.9rem;
}

.explanation-textarea {
  min-height: 20vh;
  outline: none;
  background-color: #f9f9f9;
}

/* Add this media query for smaller screens */
@media (max-width: 600px) {
  header {
    flex-direction: column;
  }

  .logo {
    margin-bottom: 1rem;
  }

  textarea,
  .explanation-textarea {
    min-height: 15vh;
  }

    .code-input,
  .code-explanation {
    text-align: center;
  }

  
}


</style>