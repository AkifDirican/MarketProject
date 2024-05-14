<template>
  <div class="page">
    <div class="left">
      <form class="sort">
        <h1> Sort </h1>
        <label><input type="radio" name="sort" value="low2High" v-model="sortOrder" @change="updateSortOrder"> Price Low to High</label>
        <br>
        <label><input type="radio" name="sort" value="high2Low" v-model="sortOrder" @change="updateSortOrder"> Price High to Low</label>
      </form>
      <form class="brandSearch">
        <h1> Brands </h1>
        <input placeholder="Search Brand" v-model="brandSearchInput"></input>
        <br>
        <label><input type="checkbox" v-model="selectAllBrands"> Select All</label>
        <br>
        <label v-for="(brand, index) in filteredBrands" :key="index">
          <input type="checkbox" v-model="selectedBrands" :value="brand"> {{ brand }}
          <br>
        </label> 
      </form>
      <form class="tagSearch">
        <h1> Tags </h1>
        <input placeholder="Search Tags" v-model="tagSearchInput"></input>
        <br>
        <label><input type="checkbox" v-model="selectAllTags"> Select All</label>
        <br>
        <label v-for="(tag, index) in filteredTags" :key="index">
          <input type="checkbox" v-model="selectedTags" :value="tag"> {{tag}}
          <br>
        </label>
      </form>
    </div>
    <div class="mid">
      <div class="squares">
        <div v-for="(item, index) in itemsOnPage" :key="index" class="square"> 
          <p>{{ item.price }}</p>
          <p>{{ item.tags }}</p>
          <p>{{ item.manufacturer }}</p>
          <button @click="addToSelectedItems(item)">Add</button>
        </div>
      </div>
      <div class="pagination">
        <!-- Previous Ten Pages Button -->
        <button @click="moveBack(10)"> << </button>
        <!-- Previous Page Button -->
        <button @click="moveBack(1)"> < </button>
        <!-- Page Buttons -->
        <button v-for="pageNumber in visiblePages" :key="pageNumber" @click="changePage(pageNumber)">
          {{ pageNumber }}
        </button>
        <!-- Next Page Button -->
        <button @click="moveForward(1)"> > </button>
        <!-- Next Ten Pages Button -->
        <button @click="moveForward(10)"> >></button>
      </div>
    </div>
    <div class="right"> 
      <div class="bag">
          <div v-if="selectedItems.size === 0">Your bag is empty. </div>
          <div v-else>
            <ul>
              <li v-for="([item, quantity], index) in selectedItems" :key="index"> {{ item.manufacturer }} - {{ item.price }} ({{ quantity }}) 
              <br></li>  
            </ul>
            <br>
            <p> Total Price: {{ calculateTotalPrice() }}</p>
          </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import items from "@/components/items_2.json";

// Brand and Tag Functions
// All Brands and Tags that go into checkboxes 
const brands = computed(() => [...new Set(items.map(item => item.manufacturer))]);
const tags = computed(() => [...new Set(items.flatMap(item => item.tags))]);

// Selected brands and tags
const selectedBrands = ref([]);
const selectAllBrands = ref(false);
const selectedTags = ref([]);
const selectAllTags = ref(false);

const brandSearchInput = ref('');
const tagSearchInput = ref('');

// Filter brands based on search input
const filteredBrands = computed(() => {
  const searchInput = brandSearchInput.value.toLowerCase();
  return brands.value.filter(brand => brand.toLowerCase().startsWith(searchInput));
});

const filteredTags = computed(() => {
  const searchInput = tagSearchInput.value.toLowerCase();
  return tags.value.filter(tag => tag.toLowerCase().startsWith(searchInput));
});

// Checks wheter select all checkbox is selected, if so selected brands/tags becomes all the brands or tags
watch([selectAllTags, selectAllBrands], ([newTagsValue, newBrandsValue]) => {
  if (newTagsValue) {
    selectedTags.value = [...tags.value];
  }
  if (newBrandsValue) {
    selectedBrands.value = [...brands.value];
  }
});

// Filter items based on selected brands and tags
const filteredItems = computed(() => {
  let filtered = items;

  if (selectedBrands.value.length > 0) {
    filtered = filtered.filter(item => selectedBrands.value.includes(item.manufacturer));
  }
  if (selectedTags.value.length > 0) {
    filtered = filtered.filter(item => item.tags.some(tag => selectedTags.value.includes(tag)));
  }

  return filtered;
});

// Sort Function
const sortOrder = ref(''); // Initial sort order

const updateSortOrder = (event) => {
  sortOrder.value = event.target.value;
};

const sortedItems = computed(() => {
  const sortedArray = [...filteredItems.value];

  if (sortOrder.value === 'low2High') {
    sortedArray.sort((a, b) => a.price - b.price);
  } else if (sortOrder.value === 'high2Low') {
    sortedArray.sort((a, b) => b.price - a.price);
  }

  return sortedArray;
});

// MID FUNCTIONS - Pagination and Showing items

const itemsPerPage = 32; // Number of items per page
let currentPage = ref(1); // Current page index

// Calculate total number of pages, start index and end index
const totalPages = computed(() => Math.ceil(filteredItems.value.length / itemsPerPage));
const startIndex = computed(() => (currentPage.value - 1) * itemsPerPage);
const endIndex = computed(() => Math.min(startIndex.value + itemsPerPage, filteredItems.value.length));

//Calculate the items on page to be shown
const itemsOnPage = computed(() => sortedItems.value.slice(startIndex.value, endIndex.value));

// Determine the visible page numbers
const visiblePages = computed(() => {
  const totalPagesValue = totalPages.value;
  const currentPageValue = currentPage.value;
  const range = [];
  let start = Math.max(1, currentPageValue - 2);
  let end = Math.min(totalPagesValue, start + 4);
  while (start <= end) {
    range.push(start);
    start++;
  }
  return range;
});

// Method to change current page
const changePage = (page) => {
  currentPage.value = page;
};

// Method to move to backwards
const moveBack = (back) => {
  currentPage.value = Math.max(currentPage.value - back, 1);
};

// Method to move forward
const moveForward = (forw) => {
  currentPage.value = Math.min(currentPage.value + forw, totalPages.value);
};

// Right Functions
const selectedItems = ref(new Map());

const addToSelectedItems = (item) => {
  const quantity = selectedItems.value.get(item) || 0;
  selectedItems.value.set(item, quantity + 1); };

const calculateTotalPrice = () => {
  let totalPrice = 0;
  for (const [item, quantity] of selectedItems.value.entries()) {
    totalPrice += item.price * quantity;
  }
  return totalPrice; };

</script>

<style scoped>
.left{
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-start;
}

.right{
  width: 200px;
  justify-content: flex-start;
  align-items: flex-start;
  background-color: rgb(180,210,290);
}

.mid {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.page{
  display: flex;
  flex-direction: row;
  width: 1500px;
  gap: 20px;
}

.sort, .brandSearch, .tagSearch{
  width: 200px;
  height: 150px;
}

.sort{ 
  height: 100px;
  background-color: rgb(110, 110, 220);
  padding-left: 5px;}

.brandSearch{ 
  background-color: rgb(110, 220, 110);
  height: 150px; 
  overflow: auto;
  padding-left: 5px;}

.tagSearch{ 
  background-color: rgb(220, 110, 110);
  height: 150px; 
  overflow: auto;
  padding-left: 5px;}

.squares{
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: space-evenly;
  gap: 10px;

  width: 1000px;
  height: 600px;

  border: solid;
  border-width: 3px;
  border-color: black;
  overflow: auto;
}
.square{
  padding-left: 5px;

  background-color:pink;
  width: 220px;
  height: 150px;
  margin: 5px 5px;

  border:solid;
  border-color:black;
}
</style>