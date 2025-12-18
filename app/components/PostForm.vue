<template>
    <div class="bg-white p-6 rounded-lg shadow-md max-w-3xl mx-auto">
      <h2 class="text-xl font-semibold mb-4">Create New Post</h2>
      <form @submit.prevent="submitPost" class="space-y-4">
  
        <!-- Title -->
        <div>
          <label class="block font-medium mb-1">Title</label>
          <input
            v-model="formData.title"
            type="text"
            maxlength="255"
            placeholder="Enter title"
            class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
            required
          />
        </div>
  
        <!-- Description -->
        <div>
          <label class="block font-medium mb-1">Description</label>
          <textarea
            v-model="formData.description"
            rows="4"
            placeholder="Enter description"
            class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
            required
          ></textarea>
        </div>
  
        <!-- Quantity and Price -->
        <div class="flex gap-4 flex-wrap">
          <div class="flex-1">
            <label class="block font-medium mb-1">Quantity</label>
            <input
              v-model.number="formData.quantity"
              type="number"
              min="1"
              placeholder="100"
              class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
              required
            />
          </div>
          <div class="flex-1">
            <label class="block font-medium mb-1">Price</label>
            <input
              v-model="formData.price"
              type="number"
              min="0"
              step="0.01"
              placeholder="2500"
              class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
              required
            />
          </div>
        </div>
  
        <!-- Location -->
        <div>
          <label class="block font-medium mb-1">Location</label>
          <input
            v-model="formData.location"
            type="text"
            maxlength="255"
            placeholder="City or region"
            class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
            required
          />
        </div>
  
        <!-- Type Post, Product, Category, Currency -->
        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
          <div>
            <label class="block font-medium mb-1">Type</label>
            <select
              v-model="formData.type_post_id"
              class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
              required
            >
              <option value="">Select Type</option>
              <option v-for="(type, index) in typeOptions" :key="index" :value="index + 1">
                {{ type }}
              </option>
            </select>
          </div>
  
          <div>
            <label class="block font-medium mb-1">Product</label>
            <select
              v-model="formData.product_id"
              class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
              required
            >
              <option value="">Select Product</option>
              <option v-for="product in cropOptions" :key="product.value" :value="product.value">
                {{ product.label }}
              </option>
            </select>
          </div>
  
          <div>
            <label class="block font-medium mb-1">Category</label>
            <select
              v-model="formData.categorie_post_id"
              class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
              required
            >
              <option value="">Select Category</option>
              <option v-for="(cat, index) in categorieOptions" :key="index" :value="index + 1">
                {{ cat }}
              </option>
            </select>
          </div>
  
          <div>
            <label class="block font-medium mb-1">Currency</label>
            <select
              v-model="formData.currency_id"
              class="w-full border rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-[#10b481]"
              required
            >
              <option value="">Select Currency</option>
              <option v-for="(cur, index) in currencyOptions" :key="index" :value="index + 1">
                {{ cur }}
              </option>
            </select>
          </div>
        </div>
  
        <!-- Image -->
        <div>
          <label class="block font-medium mb-1">Image</label>
          <input type="file" accept="image/*" @change="handleImageUpload" class="w-full"/>
          <img v-if="imagePreview" :src="imagePreview" class="mt-2 w-32 h-32 object-cover rounded-md"/>
        </div>
  
        <!-- Submit -->
        <div class="text-right">
          <button
            type="submit"
            class="bg-[#10b481] text-white px-6 py-2 rounded-lg font-medium hover:bg-[#0da066] transition-colors"
          >
            Create Post
          </button>
        </div>
  
      </form>
    </div>
  </template>
  
  <script setup>
  import { ref } from "vue";
  import { API_URL } from "~/utils/config";
  
  const typeOptions = ["SELLING", "BUYING"];
  const cropOptions = ref([]);
  const categorieOptions = ["Vegetables", "Fruits", "Grains"]; // Exemple, tu peux charger depuis API
  const currencyOptions = ref([]);
  
  const formData = ref({
    title: "",
    description: "",
    quantity: null,
    price: null,
    location: "",
    image: null,
    type_post_id: null,
    product_id: null,
    categorie_post_id: null,
    currency_id: null,
  });
  
  const imagePreview = ref(null);
  
  const handleImageUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      formData.value.image = file;
      const reader = new FileReader();
      reader.onload = (ev) => (imagePreview.value = ev.target.result);
      reader.readAsDataURL(file);
    }
  };
  
  // Soumettre le formulaire
  const submitPost = async () => {
    try {
      const token = localStorage.getItem("access_token");
      if (!token) {
        alert("You must be logged in to create a post.");
        return;
      }
  
      const data = new FormData();
      Object.entries(formData.value).forEach(([key, value]) => {
        if (value !== null) data.append(key, value);
      });
  
      const res = await fetch(`${API_URL}/api/posts/`, {
        method: "POST",
        headers: {
          Authorization: `Bearer ${token}`,
        },
        body: data,
      });
  
      if (!res.ok) {
        const err = await res.json();
        console.error("Failed:", err);
        alert("❌ Failed to create post");
        return;
      }
  
      const response = await res.json();
      alert("🎉 Post created successfully!");
      console.log("Created post:", response);
    } catch (err) {
      console.error(err);
      alert("❌ Error creating post");
    }
  };
  </script>
  