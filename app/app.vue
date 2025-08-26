<!-- app.vue -->
<template>
  <div class="bg-gray-900 min-h-screen text-white flex flex-col items-center justify-center p-4">
    <div class="w-full max-w-2xl bg-gray-800 rounded-xl shadow-lg p-8 space-y-6">
      <h1 class="text-4xl font-bold text-center text-cyan-400">AI Image Analyzer</h1>
      <p class="text-center text-gray-300">Unggah sebuah gambar dan ajukan pertanyaan tentang isinya(Kami akan meroasting juga jika anda mengupload gambar oshi/waifu dan husbu kalian.).</p>

      <div class="space-y-4">
        <label for="file-upload" class="cursor-pointer bg-gray-700 hover:bg-gray-600 text-white font-bold py-3 px-4 rounded-lg inline-block w-full text-center">
          {{ selectedFile ? selectedFile.name : 'Pilih Berkas' }}
        </label>
        <input id="file-upload" type="file" @change="handleFileChange" accept="image/png,image/jpeg,image/webp" class="hidden" />
      </div>

      <div class="space-y-1">
        <label for="custom-prompt" class="text-sm font-medium text-gray-400">Prompt Kustom (Opsional)</label>
        <textarea
          id="custom-prompt"
          v-model="customPrompt"
          rows="3"
          class="w-full bg-gray-700 text-white p-3 rounded-lg focus:outline-none focus:ring-2 focus:ring-cyan-500 transition resize-y text-sm placeholder:text-gray-500"
          placeholder="Contoh: Jelaskan apa yang terjadi di gambar ini, atau buat cerita pendek tentangnya."
        ></textarea>
      </div>

      <button
        @click="processFile"
        :disabled="!selectedFile || isLoading"
        class="w-full bg-cyan-500 hover:bg-cyan-600 disabled:bg-gray-500 disabled:cursor-not-allowed text-white font-bold py-3 px-4 rounded-lg transition-colors duration-300"
      >
        <span v-if="isLoading">Sedang Memproses... ✨</span>
        <span v-else>Proses Berkas</span>
      </button>

      <div v-if="error" class="text-red-400 text-center bg-red-900/50 p-3 rounded-lg">
        {{ error }}
      </div>

      <div v-if="generatedText" class="mt-8 p-4 bg-gray-700/50 rounded-lg">
        <h2 class="text-2xl font-semibold text-center mb-4">Konten yang Dihasilkan</h2>
        <pre class="whitespace-pre-wrap text-gray-200 bg-gray-900/50 p-4 rounded-md font-mono text-sm">{{ generatedText }}</pre>
      </div>
    </div>
    <footer class="text-center text-gray-500 mt-8">
      Make with ❤ by
      <a href="https://github.com/StevanusAndika" target="_blank" rel="noopener noreferrer" class="font-semibold text-white hover:text-cyan-400 transition-all duration-200 active:scale-95 inline-block transform">Stevanus Andika</a>
    </footer>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const selectedFile = ref(null);
const isLoading = ref(false);
const generatedText = ref('');
const error = ref('');
const customPrompt = ref('');

const handleFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    selectedFile.value = file;
    generatedText.value = '';
    error.value = '';
  }
};

const processFile = async () => {
  if (!selectedFile.value) {
    error.value = 'Silakan pilih berkas terlebih dahulu.';
    return;
  }

  isLoading.value = true;
  error.value = '';
  generatedText.value = '';

  const formData = new FormData();
  formData.append('file', selectedFile.value);
  if (customPrompt.value) {
    formData.append('prompt', customPrompt.value);
  }

  try {
    const response = await fetch('/api/generate', {
      method: 'POST',
      body: formData,
    });

    if (!response.ok) {
      const errData = await response.json();
      throw new Error(errData.statusMessage || 'Gagal memproses berkas.');
    }

    const data = await response.json();
    generatedText.value = data.generatedText;

  } catch (e) {
    error.value = e.message;
  } finally {
    isLoading.value = false;
  }
};
</script>

<style>
/* Adding a style for the <pre> tag to make it look nice */
pre {
  white-space: pre-wrap;       /* Since CSS 2.1 */
  white-space: -moz-pre-wrap;  /* Mozilla, since 1999 */
  white-space: -pre-wrap;      /* Opera 4-6 */
  white-space: -o-pre-wrap;    /* Opera 7 */
  word-wrap: break-word;       /* Internet Explorer 5.5+ */
}
</style>
