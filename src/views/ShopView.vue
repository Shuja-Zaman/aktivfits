<template>
  <div class="p-5 min-h-screen bg-slate-100">
    
    <!-- Search Input -->
    <div class="mb-4 flex justify-between items-center">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search by product ..."
        class="p-2 border-[1px] focus:border-zinc-700 border-zinc-400 outline-none focus:outline-none rounded-md w-full"
      />
      
      <!-- Category Menu -->
      <select v-model="selectedCategory" class="p-2 border border-gray-300 rounded-md ml-4">
        <option value="">All Categories</option>
        <option v-for="category in categories" :key="category.id" :value="category.id">
          {{ category.name }}
        </option>
      </select>
    </div>

    <!-- Error Message -->
    <div v-if="error" class="text-center py-4 text-red-500">
      <p>Error fetching products: {{ error }}</p>
    </div>

    <div class="">

      <button @click="showBundles = !showBundles" class="text-slate-100  font-bold bg-gradient-to-r from-orange-400 to-orange-600 hover:to-orange-400 mx-auto block w-full my-3 p-2 rounded">Check Special Offers</button>
      <!-- bundle -->
      <div v-if="!loading && !error && showBundles" class="gap-4 my-3 class grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4">
        <BundleCard
          v-for="bundle in bundles"
          :key="bundle.id"
          :id="bundle.id"
          :image="bundle.image"
          :name="bundle.name"
          :price="bundle.price"
        />
    </div>
      <!-- Product Grid -->
      <div v-if="!loading && !error" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
      <ProductCard
        v-for="product in filteredProducts"
        :key="product.id"
        :id="product.id"
        :image="product.imageUrls || []"
        :name="product.name"
        :price="product.price"
        :size="product.size"
        :category="product.categoryName"
      />
    </div>
    </div>

    <div v-if="loading" class="fixed inset-0 flex items-center justify-center bg-gray-100 bg-opacity-50 z-50">
      <div class="w-12 h-12 border-4 border-t-4 border-gray-200 border-t-zinc-700 rounded-full animate-spin"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { getDocs, collection, doc, getDoc } from 'firebase/firestore';
import { db } from '@/firebase/firebase';
import ProductCard from '../components/product/productCard.vue';
import BundleCard from '@/components/bundle/BundleCard.vue';

// Define state variables
const products = ref([]);
const categories = ref([]);
const bundles = ref([]);
const loading = ref(true);
const error = ref(null);
const searchQuery = ref('');
const selectedCategory = ref('');

const showBundles = ref(false);

// Fetch products and resolve category references
const fetchProducts = async () => {
  try {
    const querySnapshot = await getDocs(collection(db, 'products'));
    products.value = await Promise.all(querySnapshot.docs.map(async (doc) => {
      const productData = doc.data();
      const categoryRef = productData.categoryId;

      // Fetch the category document using the reference
      const categoryDoc = await getDoc(categoryRef);
      const categoryName = categoryDoc.exists() ? categoryDoc.data().name : 'Unknown Category';


      return {
        id: doc.id,
        ...productData,
        categoryName,
      };
    }));
  } catch (err) {
    error.value = err.message || 'Failed to fetch products';
  } finally {
    loading.value = false;
  }
};

// Fetch categories from Firestore
const fetchBundles = async () => {
  try {
    const querySnapshot = await getDocs(collection(db, 'bundles'));
    bundles.value = querySnapshot.docs.map(doc => ({
      id: doc.id,
      ...doc.data(),
    }));
    // console.log(bundles.value)
  } catch (err) {
    error.value = err.message || 'Failed to fetch categories';
  }
};

// Fetch categories from Firestore
const fetchCategories = async () => {
  try {
    const querySnapshot = await getDocs(collection(db, 'categories'));
    categories.value = querySnapshot.docs.map(doc => ({
      id: doc.id,
      name: doc.data().name,
    }));
  } catch (err) {
    error.value = err.message || 'Failed to fetch categories';
  }
};

// Computed property for filtering products
const filteredProducts = computed(() => {
  return products.value.filter(product => {
    const matchesSearch = product.name.toLowerCase().includes(searchQuery.value.toLowerCase());
    const matchesCategory = selectedCategory.value ? product.categoryId.id === selectedCategory.value : true;
    return matchesSearch && matchesCategory;
  });
});

// Fetch products and categories on component mount
onMounted(() => {
  fetchProducts();
  fetchCategories();
  fetchBundles();
});
</script>

<style scoped>
/* Scoped styles are omitted since we're using Tailwind CSS classes directly */
</style>
