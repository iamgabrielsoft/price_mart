<template>
  <v-container >
    <div class="d-flex align-center justify-center pt-8 pb-1" >
     <v-icon icon="mdi-cart" color="orange" size="60"/>
      <h1 class="text-center ml-4" style="font-size: 5rem;">Price_Mart</h1>
    </div>


    <p class="text-center mb-16" style="line-height: 20px; font-size: 0.9rem; margin-inline: 14rem;">
      This price checker app is a great way to save money and make informed purchasing decisions. 
      It is convenient and easy to use, and it can help you find the best deals on your everyday purchases.
    </p>
    <div class="d-flex align-center mx-auto justify-center" >
   
      <div style="width: 50%;">
        <v-file-input 
          v-model="fileInput" 
          label="Upload image file" 
          :show-size="1000"
          prepend-icon="mdi-camera"
          accept="image/png, image/jpeg"
          :rules="rules"
          @keyup.enter="loadInformation(fileInput)"
        />
      </div>
      <v-btn class="ml-4 mb-4" density="default" size="x-large" 
       prepend-icon="mdi-check-circle" @click="loadInformation(fileInput)">
        <template v-slot:prepend>
          <v-icon color="orange"></v-icon>
        </template>
        Go 
      </v-btn>
    </div>

    <!-- when loading -->

    <div class="text-left" style="text-overflow: ellipsis; hyphens: auto; line-height: 1.2rem;">
      <div class="d-flex my-4">
        <v-chip size="default" color="orange" class="mb-5">market place</v-chip>
        <v-chip size="default" color="orange" class="mb-5 ml-3">prompt command</v-chip>
        <v-chip size="default" color="orange" class="mb-5 ml-3">result</v-chip>
      </div>

      <p style="font-size: 1rem;">{{  text }}</p>
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
  const base64EncodedDataPromise = new Promise((resolve) => {
    const reader = new FileReader();
    reader.onloadend = () => resolve(reader.result.split(",")[1]);
    reader.readAsDataURL(file);
  });


  return {
    inlineData: { data: await base64EncodedDataPromise, mimeType: file.type },
  };
}

const loadInformation = async (fileInput) => {
  let result; 

  isLoading.value = true; 

  try {

    const ai = new GoogleGenerativeAI(import.meta.env.VITE_GEMINI_TOKEN); 
    const model = ai.getGenerativeModel({ 
      model: 'gemini-pro-vision', 
      generationConfig: { temperature: 0,  maxOutputTokens: 100 }, 
    }); 

    result = await model.generateContent([
      `
      If this image is a list of food items, using data from the National Bureau of
      Statistics in Nigeria, list out the food items in this image, their sizes/quantities
      and their current prices. If it is a food item, mention its quantity and current price.
      `,
      {
        inlineData: (await fileToGenerativePart(fileInput)).inlineData,
      }
    ]); 

    console.log(result.response);

   text.value = result.response.text(); 
  }catch(error){
    text.value = error.message;
  }finally {
    isLoading.value = false;
  } 
}; 



</script>
