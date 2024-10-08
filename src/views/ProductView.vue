<template>
  <div class="p-5 min-h-screen bg-slate-100">
    <div v-if="loading" class="fixed inset-0 flex items-center justify-center bg-gray-100 bg-opacity-50 z-50">
      <div class="w-12 h-12 border-4 border-t-4 border-gray-200 border-t-zinc-700 rounded-full animate-spin"></div>
    </div>

    <div v-if="!loading && product" class="container mx-auto bg-white shadow-md rounded-lg overflow-hidden flex flex-col lg:flex-row">
      <!-- Product Image Gallery -->
      <div class="w-full lg:w-[25%] p-4">
        <!-- Main Image -->
        <img :src="selectedImage" :alt="product.name" class="border-zinc-300 border-[1px] w-full img rounded-lg" />

        <!-- Thumbnails -->
        <div class="flex gap-2 mt-4">
          <img
            v-for="(image, index) in product.imageUrls"
            :key="index"
            :src="image"
            :alt="`Thumbnail ${index + 1}`"
            @click="selectedImage = image"
            class="w-16 h-16 object-cover border-2 border-gray-200 rounded-lg cursor-pointer"
          />
        </div>
      </div>

      <!-- Product Details -->
      <div class="w-full p-4 flex flex-col justify-between">
        <div>
          <h1 class="text-3xl kanit-extrabold mb-2">{{ product.name }}</h1>
          <p class="kanit-thin mb-2">{{ product.description }}</p>
          <p class="kanit-thin mb-2"><span class="kanit-medium">Price</span>: Rs. {{ product.price }}</p>

          <!-- Size Selector -->
          <div v-if="sizeArray.length" class="flex items-center gap-5">
            <p class="mb-2">Size:</p>
            <div class="flex gap-4">
              <label v-for="size in sizeArray" :key="size" class="flex items-center gap-2">
                <input type="radio" v-model="selectedSize" :value="size" class="form-radio h-5 w-5" />
                <span class="border-[1px] py-1 px-2 rounded-sm bg-zinc-200">{{ size }}</span>
              </label>
            </div>
          </div>
        </div>

        <!-- Quantity Selector -->
        <div class="flex flex-row items-center gap-3 mt-2">
          <label for="quantity" class="mb-2">Quantity:</label>
          <input
            id="quantity"
            type="number"
            v-model="quantity"
            min="1"
            class="border border-zinc-300 rounded-md p-2 md:w-20 sm:w-10 focus:outline-none focus:border-zinc-600"
          />
        </div>

        <p v-if="message" class="text-red-400 kanit-thin mt-2">{{ message }}!</p>

        <!-- Add to Cart Button -->
        <Button @click="addToCart" name="Add to Cart" class="mt-8" />
      </div>
      
    </div>
    <img v-if="category == 'KZN9CkQxyPbH35BlC6W8' " src="../assets/images/t-shirt.png" alt="Chart" class="lg:w-[75%] mx-auto my-10">
  <img v-if="category == 'rmQ30Tb2jyyZRIFQA7ot' " src="../assets/images/oversize_t-shirt.png" alt="Chart" class="lg:w-[75%] mx-auto my-10">
  <img v-if="category == 'xlclnMgVtuhLgy8gAH10' " src="../assets/images/trouser.png" alt="Chart" class="lg:w-[75%] mx-auto my-10">
  </div>
</template>


<script setup>
import { ref, onMounted, computed } from 'vue';
import { useRoute } from 'vue-router';
import { getDoc, doc } from 'firebase/firestore';
import { db } from '@/firebase/firebase'; // Adjust the import path as needed
import Button from '@/components/button/Button.vue';
import { useCartStore } from '@/store/cart.js';

const route = useRoute();
const productId = route.params.id;
const product = ref(null);
const loading = ref(true); // Loading state
const quantity = ref(1); // Initialize quantity to 1
const selectedSize = ref(''); // To hold the selected size
const cartStore = useCartStore();

const category = ref('');

const message = ref('');

const validateQuantity = () => {
  const value = quantity.value;

  // Check if the value is empty or not a number
  if (isNaN(value) || value === '') {
    message.value = 'Please enter a valid number.';
    return false;
  }

  // Ensure the value is between 1 and 10 (inclusive)
  if (value < 1 || value > 10) {
    message.value = 'Quantity must be between 1 and 10.';
    return false;
  }

  // Clear the message if the value is valid
  message.value = '';
  return true;
};



// Computed property to derive the size array from product.size
const sizeArray = computed(() => product.value?.size ? product.value.size.split(',').map(s => s.trim()) : []);

const selectedImage = ref(''); // Initializing selectedImage

const fetchProduct = async () => {
  try {
    const docRef = doc(db, 'products', productId);
    const docSnap = await getDoc(docRef);
    
    if (docSnap.exists()) {
      product.value = docSnap.data();
      if (product.value.imageUrls && product.value.imageUrls.length > 0) {
        selectedImage.value = product.value.imageUrls[0]; // Initialize with the first image
      }
    } else {
      console.error('No such document!');
    }
    category.value = product.value.categoryId.id;
  } catch (err) {
    console.error('Error fetching product:', err);
  } finally {
    loading.value = false;
  }
};


const addToCart = () => {
  if (!selectedSize.value && sizeArray.value.length) {
    message.value = 'Please select a Size'; 
    return;
  }

  message.value = '';

  if (validateQuantity()) {
    const productToAdd = {
      id: productId,
      imgUrl: product.value.imageUrls[0],
      name: product.value.name,
      price: product.value.price,
      size: selectedSize.value,
      quantity: quantity.value,
    };


  cartStore.addProduct(productToAdd);

  alert("Prodcut added to Cart.")
  }
};

// Fetch product details when component is mounted
onMounted(() => {
  fetchProduct();
});
</script>

<style scoped>
/* Scoped styles are omitted since we're using Tailwind CSS classes directly */

.img{
  background-size: cover;
  object-fit: contain;
}

</style>
