<template>
  <div class="p-5 min-h-screen flex">
    <div class="border-r-[1px] pr-2 w-[25%]">
        <Message/>
        <AddBundles/>
    </div>
    <div class="w-[50%] pl-5 border-r-[1px] pr-2">
        <Order />
    </div>
    <div class="w-[25%] pl-5">
        <CreateCategory/>
        <CreateProduct class="mt-3"/>
    </div>
  </div>
  <div class="flex">  
    <FetchProducts class="border-r-[1px]"/>
    <FetchCategories/>
  </div>
  <div>
    <FetchBundles/>
  </div>
</template>

<script setup>
import { onMounted } from 'vue';
import { useRouter } from 'vue-router';

import CreateCategory from '@/components/adminComponents/CreateCategory.vue';
import CreateProduct from '@/components/adminComponents/CreateProduct.vue';
import Message from '@/components/adminComponents/Message.vue';
import Order from '@/components/adminComponents/Order.vue';
import FetchProducts from '@/components/adminComponents/FetchProducts.vue';
import FetchCategories from '@/components/adminComponents/FetchCategories.vue';
import AddBundles from '@/components/adminComponents/AddBundles.vue';
import FetchBundles from '@/components/adminComponents/FetchBundles.vue';

const router = useRouter();

const checkAuth = () => {
  const email = localStorage.getItem('email'); // Use 'email' as the key
  if (email !== import.meta.env.VITE_EMAIL_COMP) {
    router.push('/404'); // Redirect to home if the user is not authenticated
  }
}

onMounted(() => {
  checkAuth();
});

</script>

<style>
/* Add your styles here if needed */
</style>
