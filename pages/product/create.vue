<script lang="ts" setup>
import { useCategoryStore } from '~/stores/category';
import { useStorageStore } from "~/stores/storage";
import { useProductsStore } from '~/stores/products';
import type { FormField } from '~/types/products';

const categoryStore = useCategoryStore();
const storageStore = useStorageStore();
const productStore = useProductsStore();

const isSuccess = ref(false);
const isShowAlert = ref(false);
const message = ref("");
const router = useRouter();
const isLoading = ref(false);

const { categories } = storeToRefs(categoryStore);
categoryStore.getAllCategory();

const formCreateProduct = [
  { name: "name", label: "Nama", type: "text", value: "", placeholder: "contoh: Abdul", required: true },
  { name: "description", label: "Alasan/jam masuk", type: "text", value: "", placeholder: "contoh: 07:00/jika sakit beri tahu sakit apa", required: true },
  { name: "price", label: "Absen", type: "number", value: "", placeholder: "contoh: 01", required: true },
  { name: "image", label: "Foto Kamu", type: "file", value: {}, placeholder: "", required: true },
  { name: "category", label: "Keterangan", type: "select", value: "", placeholder: "pilih jam jika telat", required: true },
];

const createProduct = async () => {
  isLoading.value = true;
  let file = formCreateProduct.find((item) => item.type === "file");
  file = await uploadFile(file?.value);
  const result: { [key: string]: any } = {};
  formCreateProduct.forEach((item: FormField) => {
    if(item.name){
      result[item.name] = item.value;
    }
  })
  result.image = file;
  await productStore.createProduct(result);
  if(!productStore.status){
    isSuccess.value = productStore.status;
    message.value = productStore.message;
    isShowAlert.value = true;
    isLoading.value = false;
    window.scrollTo({
      top: 0,
      behavior: "smooth",
    });
  }else{
    isSuccess.value = productStore.status;
    message.value = productStore.message;
    isShowAlert.value = true;
    isLoading.value = false;
    window.scrollTo({
      top: 0,
      behavior: "smooth",
    });
    setTimeout(() => {
      router.push({ path: "/product" });
    }, 1000);
  }
}

const uploadFile = async (formFile: any) => {
  const file = formFile?.target?.files[0];
  const payload = new FormData();
  payload.append("file", file);
  const pathname = `${Date.now().toString()}.${file.type.split("/")[1]}`;
  await storageStore.uploadFile(pathname, payload);
  if(storageStore.status){
    return storageStore.storage.Key;
  }
}
</script>



<template>
  <section class="flex justify-center py-10">
    <div class="w-[500px]">
      <h1 class="text-2xl mb-7 font-medium">Create Products</h1>
      <div v-if="isShowAlert" :class="`p-4 mb-4 text-sm rounded-lg ${isSuccess ? 'bg-green-100 text-green-800' : 'text-red-800 bg-red-100'}`" role="alert">
        {{ message }}
      </div>
      <form @submit.prevent="createProduct">
        <div class="mb-6" v-for="(item, index) in formCreateProduct" :key="index">
          <label
            :for="item.name"
            class="block mb-2 text-sm font-medium text-gray-900"
          >
            {{ item.label }}
          </label>
          <select
            v-if="item.type === 'select'"
            :id="item.name"
            class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
            v-model="item.value"
          >
            <option disabled value=""></option>
            <option v-for="(category, index) in categories" :key="index" :value="category.name">{{ category.name }}</option>
          </select>
          <input
            v-else-if="item.type === 'file'"
            :type="item.type"
            :id="item.name"
            @change="item.value = $event"
            class="w-full text-sm text-gray-900 border border-gray-300 rounded-lg cursor-pointer bg-gray-50 focus:outline-none file:bg-primary file:border-none file:px-3 file:py-2 file:text-white file:mr-3"
            :required="item.required"
          >
          <input
            v-else
            :type="item.type"
            :id="item.name"
            v-model="item.value"
            class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
            :placeholder="item.placeholder"
            :required="item.required"
          >
        </div>
        <button type="submit" class="text-white bg-primary hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm w-full sm:w-auto px-5 py-2.5 text-center">
          <span v-if="!isLoading">Absen Sekarang</span>
          <div v-else class="flex items-center gap-3">
              <div class="w-5 h-5 rounded-full border-2 border-t-blue-500 animate-spin"></div>
              <span>Loading</span>
          </div>
        </button>
      </form>
    </div>    
  </section>
</template>