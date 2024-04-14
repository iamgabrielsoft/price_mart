<template>
  <v-container>
    <h1 class="text-center mb-8">Price_Mart</h1>
    <p class="text-center mb-16 mx-16">
      This price checker app is a great way to save money and make informed purchasing decisions. 
      It is convenient and easy to use, and it can help you find the best deals on your everyday purchases.
    </p>
    <div class="d-flex align-center" style="margin-inline: 4rem;">
      <!-- <v-img src="@/assets/logo.png" height="40px" /> -->

      <v-file-input 

        v-model="fileInput" 
        label="Upload image file" 
        :show-size="1000"
        prepend-icon="mdi-camera"
        accept="image/png, image/jpeg"
        :rules="rules"
        @keyup.enter="loadInformation(fileInput)"
      />
      <v-btn size="large" style="margin-left: 1rem;" @click="loadInformation(fileInput)">Go </v-btn>
    </div>

    <!-- when loading -->

    <div class="text-center mb-10">
      {{  text }}
    </div>

   <loading v-model:active="isLoading"/>
  </v-container>
</template>

<script setup>
import { onMounted, watchEffect, ref } from 'vue';
import { GoogleGenerativeAI } from '@google/generative-ai'; 
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/css/index.css';



const fileInput = ref(null);  
const text = ref(''); 
const isLoading = ref(false); 
const rules = [!fileInput || !fileInput.length || fileInput[0].size < 2000000 || 'File size should be less than 2 MB!']


async function fileToGenerativePart(file) {
  const base64EncodedDataPromise = new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => resolve(reader.result);
    reader.onerror = error => reject(error)

    // reader.onloadend = () => resolve(reader.result);
  });


  return {
    inlineData: { data: await base64EncodedDataPromise, mimeType: 'image/jpg' },
  };
}

const loadInformation = async (fileInput) => {
  let result; 

  isLoading.value = true; 
  try {
    const ai = new GoogleGenerativeAI(import.meta.env.VITE_GEMINI_TOKEN); 
    const model = ai.getGenerativeModel({ 
      model: 'gemini-pro', 
      generationConfig: { temperature: 0,  maxOutputTokens: 10000,}, 

    }); 

    result = await model.generateContent([
      `
        We are expecting one of two types of images. It is either a list of items or single food item. 
        If it is a list, write the list of items and their estimated prices using Nigeria's current inflation rate. 
        If it is a single food item, determine the size and give the price estimate
      `,
      fileInput,

      {
        inlineData: fileToGenerativePart(fileInput),

      }
    ]); 

    text.value = result.response.text(); 
  }catch(error){
    text.value = error.message;
  }finally {
    isLoading.value = false;
  } 
}; 



</script>
