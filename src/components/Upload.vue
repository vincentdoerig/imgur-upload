<template>
  <div
    class="flex flex-col justify-center max-w-lg mx-auto mt-6 space-y-4 align-center sm:pt-5"
  @paste="addFileTroughPaste">
    <div
      @drop.prevent="addFile"
      @dragover.prevent
      @dragover="dragging = true"
      @drop="dragging = false"
      @dragleave="dragging = false"
      :class="[dragging ? 'bg-indigo-50' : '']"
      class="flex justify-center px-6 pt-5 pb-6 mt-2 border-2 border-gray-300 border-dashed rounded-md bg-gray-50"
    >
      <div v-if="!file" class="text-center">
        <svg
          class="w-12 h-12 mx-auto text-gray-400"
          stroke="currentColor"
          fill="none"
          viewBox="0 0 48 48"
        >
          <path
            d="M28 8H12a4 4 0 00-4 4v20m32-12v8m0 0v8a4 4 0 01-4 4H12a4 4 0
              01-4-4v-4m32-4l-3.172-3.172a4 4 0 00-5.656 0L28 28M8 32l9.172-9.172a4
              4 0 015.656 0L28 28m0 0l4 4m4-24h8m-4-4v8m-12 4h.02"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
        <p class="mt-1 text-sm text-gray-600">
          <label
            class="font-medium text-indigo-600 transition duration-150 ease-in-out cursor-pointer hover:text-indigo-500 focus:outline-none focus:underline"
          >
            Upload a file
            <input
              type="file"
              @change="addFileTroughInput"
              class="fixed hidden"
              name="image"
            />
          </label>
          or drag and drop
        </p>
        <p class="mt-1 text-xs text-gray-500">
          PNG, JPG, GIF up to 10MB
        </p>
      </div>
      <div v-else class="flex flex-col items-center space-y-3">
        <h2 class="mb-2 text-2xl">Ready to upload</h2>
        <img v-if="srcFile" :src="srcFile" class="border-indigo-200 border-dashed border-3 max-h-64" alt="Image to upload">
        <div class="inline-flex items-center">
          <svg class="w-4 h-4 mr-1 text-gray-700 fill-current" viewBox='0 0 512 512'>
            <path d='M416,64H96a64.07,64.07,0,0,0-64,64V384a64.07,64.07,0,0,0,64,64H416a64.07,64.07,0,0,0,64-64V128A64.07,64.07,0,0,0,416,64Zm-80,64a48,48,0,1,1-48,48A48.05,48.05,0,0,1,336,128ZM96,416a32,32,0,0,1-32-32V316.37l94.84-84.3a48.06,48.06,0,0,1,65.8,1.9l64.95,64.81L172.37,416Zm352-32a32,32,0,0,1-32,32H217.63L339.05,294.58a47.72,47.72,0,0,1,61.64-.16L448,333.84Z'/>
          </svg>
          {{ file.name }} <span class="ml-2">({{file.size | fileSize}})</span>
          <button
            @click="removeFile()"
            title="Remove"
            class="ml-3 rounded hover:bg-gray-200"
          >
            <svg fill="currentColor" viewBox="0 0 20 20" class="w-4 h-4">
              <path
                d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
                clip-rule="evenodd"
                fill-rule="evenodd"
              ></path>
            </svg>
          </button>
        </div>
        <div class="px-4 py-3 text-right sm:px-6">
          <span class="inline-flex rounded-md shadow-sm">
            <button
              type="submit"
              class="inline-flex justify-center px-4 py-2 text-sm font-medium leading-5 text-white transition duration-150 ease-in-out bg-indigo-600 border border-transparent rounded-md hover:bg-indigo-500 focus:outline-none focus:border-indigo-700 focus:shadow-outline-indigo active:bg-indigo-700"
              :disabled="uploadDisabled"
              @click="upload"
            >
              {{ uploading ? 'Uploading...' : 'Upload' }}
            </button>
          </span>
        </div>
      </div>
    </div>
    <div v-if="error" class="p-4 rounded-md bg-red-50">
      <div class="flex">
        <div class="flex-shrink-0">
          <svg
            class="w-5 h-5 text-red-400"
            fill="currentColor"
            viewBox="0 0 20 20"
          >
            <path
              fill-rule="evenodd"
              d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z"
              clip-rule="evenodd"
            />
          </svg>
        </div>
        <div class="ml-3">
          <h3 class="text-sm font-medium leading-5 text-red-800">
            {{ error }}
          </h3>
        </div>
      </div>
    </div>
    <div v-if="success" class="p-4 rounded-md bg-green-50">
      <div class="flex">
        <div class="flex-shrink-0">
          <svg
            class="w-5 h-5 text-green-400"
            fill="currentColor"
            viewBox="0 0 20 20"
          >
            <path
              fill-rule="evenodd"
              d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
              clip-rule="evenodd"
            />
          </svg>
        </div>
        <div class="ml-3">
          <h3 class="text-sm font-medium leading-5 text-green-800">
            Success
          </h3>
          <div class="mt-2 text-sm leading-5 text-green-700">
            <p>
              <a :href="successData.link">{{ successData.link }}</a>
            </p>
          </div>
          <div class="mt-4">
            <div class="-mx-2 -my-1.5 flex">
              <button
                v-if="copied"
                class="ml-3 px-2 py-1.5 inline-flex items-center rounded-md text-sm leading-5 font-medium text-green-800 hover:bg-green-100 focus:outline-none focus:bg-green-100 transition ease-in-out duration-150"
              >
                <svg
                  class="w-5 h-5 mr-2 -ml-1"
                  fill="currentColor"
                  viewBox="0 0 20 20"
                >
                  <path
                    d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                    clip-rule="evenodd"
                    fill-rule="evenodd"
                  ></path>
                </svg>
                Copied
              </button>
              <div v-else>
                <button
                  class="ml-3 px-2 py-1.5 rounded-md text-sm leading-5 font-medium text-green-800 hover:bg-green-100 focus:outline-none focus:bg-green-100 transition ease-in-out duration-150"
                  v-clipboard="successData.link"
                  v-clipboard:success="copySuccess"
                >
                  Copy link
                </button>
                <button
                  v-if="successData.mp4"
                  class="ml-3 px-2 py-1.5 rounded-md text-sm leading-5 font-medium text-green-800 hover:bg-green-100 focus:outline-none focus:bg-green-100 transition ease-in-out duration-150"
                  v-clipboard="successData.mp4"
                  v-clipboard:success="copySuccess"
                >
                  Copy MP4 link
                </button>
                <button
                  v-if="successData.gifv"
                  class="ml-3 px-2 py-1.5 rounded-md text-sm leading-5 font-medium text-green-800 hover:bg-green-100 focus:outline-none focus:bg-green-100 transition ease-in-out duration-150"
                  v-clipboard="successData.gifv"
                  v-clipboard:success="copySuccess"
                >
                  Copy GIFV link
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const allowedFormats = /video|image/;


export default {
  data: () => ({
    file: null,
    success: false,
    error: null,
    dragging: false,
    successData: null,
    uploading: false,
    copied: false,
    srcFile: null,
  }),
  computed: {
    uploadDisabled() {
      return this.file && false;
    },
  },
  filters: {
    fileSize(bytes) {
      // thank you stackoverflow
      // https://stackoverflow.com/a/14919494
      const threshold = 1000;
      if (Math.abs(bytes) < threshold) {
        return `${bytes} B`;
      }
      const units = ['kB', 'MB', 'GB'];
      let u = -1;
      do {
        // eslint-disable-next-line no-param-reassign
        bytes /= threshold;
        u += 1;
      } while (Math.abs(bytes) >= threshold && u < units.length - 1);
      return `${bytes.toFixed(1)} ${units[u]}`;
    },
  },
  methods: {
    removeFile() {
      this.error = null;
      this.file = null;
      this.srcFile = null;
      this.uploading = false;
    },
    setupReader() {
      if (this.file.type.split('/')[0] !== 'image') return;
      const reader = new FileReader();
      reader.onload = (ev) => {
        this.srcFile = ev.target.result;
      };
      reader.readAsDataURL(this.file);
    },
    addFile(e) {
      this.removeFile();
      const droppedFile = e.dataTransfer.files[0];
      // console.log(droppedFile.type.split('/')[0].test(/video|image/));
      if (!allowedFormats.test(droppedFile.type.split('/')[0])) {
        this.error = 'File type not supported';
        return;
      }
      if (!droppedFile) return;
      this.file = droppedFile;
      this.setupReader();
    },
    addFileTroughInput(e) {
      this.removeFile();
      const selectedFile = e.target.files[0];
      if (!selectedFile) return;
      if (!allowedFormats.test(selectedFile.type.split('/')[0])) {
        this.error = 'File type not supported';
        return;
      }
      this.file = selectedFile;
      this.setupReader();
    },
    addFileTroughPaste(e) {
      this.removeFile();
      const { items } = e.clipboardData;
      // eslint-disable-next-line no-plusplus
      for (let i = 0; i < items.length; i++) {
        if (items[i].type.split('/')[0] === ('image' || 'video')) {
          this.file = items[i].getAsFile();
          this.setupReader();
        }
      }
    },
    upload() {
      const formData = new FormData();
      formData.append('image', this.file);
      this.uploading = true;

      fetch('https://api.imgur.com/3/image', {
        method: 'POST',
        body: formData,
        headers: {
          Authorization: 'Client-ID e4f58fc81daec99',
        },
      })
        .then((res) => res.json())
        .then((res) => {
          if (!res.success) {
            this.success = false;
            if (typeof res.data.error === 'object' && res.data.error.message) { this.error = res.data.error.message; } else this.error = res.data.error;
            return;
          }
          this.successData = res.data;
          this.file = null;
          this.success = true;
        })
        .catch((e) => {
          this.error = e.message;
          console.error(JSON.stringify(e.message));
        })
        .finally(() => {
          this.uploading = false;
        });
    },
    copySuccess() {
      this.copied = true;
      setTimeout(() => {
        this.copied = false;
      }, 2050);
    },
  },
};
</script>
