<template>
  <div class='fullpage'>
    <div class="title"> market </div>
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
        <div class="midtop">
          <div class="midtitle">Products</div>
          <label> <button class="addtop">ADD</button> <button class="shirttop">SHIRT</button>
          </label>
        </div>
        <div class="squares">
          <div v-for="(item, index) in itemsOnPage" :key="index" class="square"> 
            <p> $ {{ item.price }}</p>
            <p>{{ item.tags }}</p>
            <p>{{ item.manufacturer }}</p>
            <p>{{ item.slug }}</p>
            <div class="buttonContainer">
              <button @click="addToSelectedItems(item)">Add</button>
            </div>
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
              <div class="price"> {{ calculateTotalPrice() }} </div>
              <br>
              <div> <button @click="remove()"> Remove All Items </button> </div>
            </div>
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

const remove = (event) => {
  selectedItems.value = new Map();
};

</script>

<style scoped>
.fullpage{
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}
.title{
  background-color: blue;
  color: white;
  font-size: 25px;
  text-align: center;
  width: 100%;
}

.left{
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-start;
  gap: 5px;
}

.right{
  width: 200px;
  justify-content: flex-start;
  align-items: flex-start;
}

.mid {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  gap: 5px;
}

.page{
  display: flex;
  flex-direction: row;
  gap: 20px;
}

h1{
  font-size: 15px;
}

p{
  font-size: 12px;
}

.sort, .brandSearch, .tagSearch{
  border: solid;
  width: 200px;
}

.sort{ 
  height: auto;
  padding: 10px;
  }

.brandSearch{ 
  height: 120px; 
  overflow: auto;
  padding: 10px;
  }

.tagSearch{ 
  height: 120px;
  overflow: auto;
  padding: 10px;}

.midtitle{
  font-size: 15px;
  font-weight:bold;
}

.squares{
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  flex-wrap: wrap;
  gap: 5px;

  width: 800px;
  height: 600px;

  overflow: auto;
}
.square{
  padding: 5px;
  display: flex;
  flex-direction: column;

  width: calc(24.5%);
  height: auto;

  box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.5);
}

.buttonContainer {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  width: 100%;
  background-color:blue;
}

.buttonContainer button{
  color:white;
  background-color:blue;
}

.bag{
  border:solid;
  border-color:blue;
  border-width:3px;
  padding: 5px;
}

.price{
  display: flex;
  flex-direction: row-reverse;
}
</style>