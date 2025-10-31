<script setup>
import { ref, onMounted, watch } from 'vue'

const currentId = ref(1)
const product = ref(null)
const loading = ref(false)
const error = ref(null)

const MAX_ID = 20 // FakeStoreAPI has 1..20
const cache = new Map()

// Definisi color palette
const COLORS = {
  primaryBlue: '#002772',
  lightPurple: '#FDE2FF',
  darkGray: '#1E1E1E',
  magenta: '#720060',
  lightBlue: '#D6E6FF',
  mediumGray: '#3F3F3F',
  lightGray: '#DCDCDC',
  white: '#FFFFFF'
}

async function fetchProduct(id) {
  loading.value = true
  product.value = null
  error.value = null
  try {
    if (cache.has(id)) {
      const cachedProduct = cache.get(id)
      // Cek apakah cached product adalah men's atau women's clothing
      const category = cachedProduct.category?.toLowerCase()
      if (category && (category.includes("men's clothing") || category.includes("women's clothing"))) {
        product.value = cachedProduct
        prefetchNext(id)
        return
      } else {
        // Skip produk yang bukan men's/women's clothing
        loading.value = false
        nextProduct()
        return
      }
    }

    const res = await fetch(`https://fakestoreapi.com/products/${id}`)
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    const data = await res.json()
    
    // Cek category sebelum menyimpan
    const category = data.category?.toLowerCase()
    if (category && (category.includes("men's clothing") || category.includes("women's clothing"))) {
      // Simpan hanya jika men's atau women's clothing
      product.value = data
      cache.set(id, data)
      prefetchNext(id)
    } else {
      // Jangan simpan, langsung skip ke produk berikutnya
      cache.set(id, data) // Simpan ke cache untuk menandai sudah dicek
      loading.value = false
      nextProduct()
      return
    }
  } catch (err) {
    error.value = err.message || String(err)
  } finally {
    setTimeout(() => { loading.value = false }, 250) // Small delay for loader visibility
  }
}

async function prefetchNext(id) {
  const next = id >= MAX_ID ? 1 : id + 1
  if (cache.has(next)) return
  try {
    const res = await fetch(`https://fakestoreapi.com/products/${next}`)
    if (!res.ok) return
    const data = await res.json()
    cache.set(next, data)
  } catch (e) {
    // Ignore prefetch errors
  }
}

function nextProduct() {
  if (loading.value) return
  currentId.value = currentId.value >= MAX_ID ? 1 : currentId.value + 1
}

onMounted(() => fetchProduct(currentId.value))
watch(currentId, id => fetchProduct(id))

function cardClass(category) {
  if (!category) return 'page-unavailable'
  const c = category.toLowerCase()
  if (c.includes('men')) return 'page-men'
  if (c.includes('women')) return 'page-women'
  return 'page-unavailable' // Fallback for other categories
}

function bgClass(category) {
  if (!category) return 'bg-unavailable'
  const c = category.toLowerCase()
  if (c.includes('men')) return 'bg-men'
  if (c.includes('women')) return 'bg-women'
  return 'bg-unavailable'
}
</script>

<template>
  <div class="viewer-root" :class="bgClass(product?.category)">
    <div class="viewer-shell">
      <div v-if="loading" class="loader-wrap">
        <article class="product-card skeleton">
          <div class="card-media">
            <div class="skeleton-img"></div>
          </div>
          <div class="card-body">
            <div class="skeleton-line short"></div>
            <div class="skeleton-line xsmall"></div>
            <div class="skeleton-line"></div>
            <div class="skeleton-line"></div>
            <div class="price-row">
              <div class="skeleton-price"></div>
              <div class="actions">
                <div class="skeleton-btn"></div>
                <div class="skeleton-btn small"></div>
              </div>
            </div>
          </div>
        </article>
      </div>

      <div v-else-if="error || !product?.category" class="unavailable-state">
        <div class="sad-face">
          <div class="eye"></div>
          <div class="eye"></div>
          <div class="mouth"></div>
        </div>
        <p class="unavailable-text">This product is unavailable to show</p>
        <button class="btn ghost" @click="nextProduct" :disabled="loading" aria-busy="loading">Next product</button>
      </div>

      <article v-else class="product-card" :class="cardClass(product.category)">
        <div class="card-media">
          <img :src="product.image" :alt="product.title" />
        </div>
        <div class="card-body">
          <h1 class="card-title">{{ product.title }}</h1>
          <div class="meta">
            <span class="category">{{ product.category }}</span>
            <div class="rating">
              {{ product.rating?.rate }}/5
              <span v-for="n in 5" :key="n" :class="['rating-bullet', { 'filled': n <= Math.round(product.rating?.rate) }]">●</span>
            </div>
          </div>
          <p class="description">{{ product.description }}</p>

          <div class="price-row">
            <div class="price">${{ product.price }}</div>
            <div class="actions">
              <button class="btn primary">Buy now</button>
              <button class="btn secondary" @click="nextProduct" :disabled="loading">Next product</button>
            </div>
          </div>
        </div>
      </article>
    </div>
    <div class="index-indicator">ID: {{ currentId }}</div>
  </div>
</template>

<style scoped>
/* Variabel CSS untuk color palette */
:root {
  --primary-blue: #002772;
  --light-purple: #FDE2FF;
  --dark-gray: #1E1E1E;
  --magenta: #720060;
  --light-blue-bg: #D6E6FF;
  --medium-gray: #3F3F3F;
  --light-gray: #DCDCDC;
  --white: #FFFFFF;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* loader (spinner) */
.loader-wrap {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 420px;
}

/* product card layout */
.product-card {
  display: flex;
  gap: 0;
  align-items: stretch;
  padding: 0;
  border-radius: 8px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15), 0 4px 12px rgba(0, 0, 0, 0.1);
  background: var(--white);
  position: relative;
  width: 70vw;
  max-width: 900px;
  min-height: 420px;
  overflow: visible;
  border: 4px solid var(--light-gray);
}

/* Background color layer behind card */
.product-card::before {
  content: '';
  position: absolute;
  top: 12px;
  left: 12px;
  right: -12px;
  bottom: -12px;
  background: var(--light-gray);
  border-radius: 8px;
  z-index: -1;
}

.card-media {
  width: 280px;
  min-width: 280px;
  height: 100%;
  min-height: 420px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--white);
  padding: 20px;
  box-sizing: border-box;
  flex-shrink: 0;
  border-right: 3px solid var(--light-blue-bg);
}

.card-media img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  display: block;
}

.card-body {
  flex: 1;
  padding: 28px 32px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  background: linear-gradient(135deg, #ffffff 0%, #2d84da 100%);
}

.card-title {
  margin: 0 0 10px 0;
  font-size: 1.25rem;
  font-weight: 600;
  color: var(--dark-gray);
  line-height: 1.4;
}

.meta {
  display: flex;
  gap: 16px;
  align-items: center;
  margin-bottom: 14px;
  font-size: 0.9rem;
}

.category {
  color: var(--medium-gray);
  text-transform: lowercase;
}

.rating {
  display: flex;
  align-items: center;
  gap: 6px;
  color: var(--medium-gray);
  font-size: 0.9rem;
}

.rating-bullet {
  font-size: 0.85rem;
  color: var(--light-gray);
}

.rating-bullet.filled {
  color: #8B4789;
}

.description {
  color: var(--medium-gray);
  line-height: 1.6;
  margin: 0 0 20px 0;
  flex: 1;
  font-size: 0.95rem;
}

.price-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: auto;
}

.price {
  font-size: 1.75rem;
  font-weight: 700;
  color: #4169E1;
}

.actions {
  display: flex;
  gap: 12px;
}

.btn {
  padding: 12px 24px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-weight: 600;
  font-size: 0.95rem;
  transition: opacity 0.2s ease;
  white-space: nowrap;
}

.btn:hover {
  opacity: 0.85;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn.primary {
  background: var(--magenta);
  color: var(--white);
}

.btn.secondary {
  background: transparent;
  color: var(--dark-gray);
  border: 2px solid var(--dark-gray);
}

/* variants per category */

/* Men's Clothing - Blue */
.page-men {
  border-color: var(--primary-blue);
}

.page-men::before {
  background: var(--primary-blue);
  opacity: 0.3;
}

.page-men .card-body {
  background: linear-gradient(135deg, #ffffff 0%, #e3f2fd 100%);
}

.page-men .card-media {
  border-right-color: var(--primary-blue);
}

.page-men .price {
  color: var(--primary-blue);
}

.page-men .btn.primary {
  background: var(--primary-blue);
  color: var(--white);
}

.page-men .btn.secondary {
  border-color: var(--primary-blue);
  color: var(--primary-blue);
}

.page-men .rating-bullet.filled {
  color: var(--primary-blue);
}

/* Women's Clothing - Magenta */
.page-women {
  border-color: var(--magenta);
}

.page-women::before {
  background: var(--magenta);
  opacity: 0.3;
}

.page-women .card-body {
  background: linear-gradient(135deg, #ffffff 0%, #fce4ec 100%);
}

.page-women .card-media {
  border-right-color: var(--magenta);
}

.page-women .price {
  color: var(--magenta);
}

.page-women .btn.primary {
  background: var(--magenta);
  color: var(--white);
}

.page-women .btn.secondary {
  border-color: var(--magenta);
  color: var(--magenta);
}

.page-women .rating-bullet.filled {
  color: var(--magenta);
}


/* Unavailable State - Gray */
.page-unavailable {
  border-color: var(--medium-gray);
}

.page-unavailable::before {
  background: var(--medium-gray);
  opacity: 0.2;
}

.page-unavailable .card-body {
  background: linear-gradient(135deg, #ffffff 0%, #f5f5f5 100%);
}

.page-unavailable .card-media {
  border-right-color: var(--medium-gray);
}

.page-unavailable .price {
  color: var(--medium-gray);
}

.page-unavailable .btn.primary {
  background: var(--medium-gray);
  color: var(--white);
}

.page-unavailable .btn.secondary {
  border-color: var(--medium-gray);
  color: var(--medium-gray);
}

.page-unavailable .rating-bullet.filled {
  color: var(--medium-gray);
}

.unavailable-state {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 20px;
  padding: 40px;
  background-color: var(--white);
  border-radius: 8px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15), 0 4px 12px rgba(0, 0, 0, 0.1);
  width: 50vw;
  max-width: 500px;
  min-height: 350px;
  border: 4px solid var(--medium-gray);
}
.unavailable-state .sad-face {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: var(--light-gray); /* Warna abu-abu yang lebih terang */
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 20px;
}
.unavailable-state .eye {
  width: 25px;
  height: 25px;
  background-color: var(--medium-gray);
  border-radius: 50%;
  position: absolute;
  top: 45px;
}
.unavailable-state .eye:first-child { left: 40px; }
.unavailable-state .eye:last-child { right: 40px; }
.unavailable-state .mouth {
  width: 80px;
  height: 40px;
  border: 8px solid var(--medium-gray);
  border-top-left-radius: 0;
  border-top-right-radius: 0;
  border-bottom-left-radius: 50px;
  border-bottom-right-radius: 50px;
  border-top: none;
  position: absolute;
  bottom: 30px;
  transform: rotate(180deg); /* Balik agar terlihat sedih */
}
.unavailable-text {
  font-size: 1.2rem;
  color: var(--dark-gray);
  margin-top: 0;
  text-align: center;
}
.unavailable-state .btn.secondary {
  border-color: var(--medium-gray);
  color: var(--medium-gray);
}

.unavailable-state .btn.secondary:hover {
  background-color: var(--light-gray);
}

/* Background solid colors */
.viewer-root {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  width: 100vw;
  padding: 40px;
  margin: 0;
  box-sizing: border-box;
  background: linear-gradient(135deg, #fbc2eb 0%, #a6c1ee 100%);
  position: fixed;
  top: 0;
  left: 0;
  overflow-y: auto;
}

.viewer-root.bg-men {
  background: linear-gradient(135deg, #fbc2eb 0%, #a6c1ee 100%);
}

.viewer-root.bg-women {
  background: linear-gradient(135deg, #fbc2eb 0%, #a6c1ee 100%);
}

.viewer-root.bg-unavailable {
  background: linear-gradient(135deg, #fbc2eb 0%, #a6c1ee 100%);
}

.viewer-shell {
  z-index: 1;
}


/* Skeleton styles */
.skeleton {
  overflow: hidden;
  border: 4px solid var(--light-gray);
}

.skeleton .skeleton-img {
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, #f0f0f0, #e0e0e0, #f0f0f0);
  background-size: 200% 100%;
  animation: shimmer 1.5s linear infinite;
}

.skeleton .skeleton-line {
  height: 14px;
  background: linear-gradient(90deg, #f0f0f0, #e0e0e0, #f0f0f0);
  background-size: 200% 100%;
  animation: shimmer 1.5s linear infinite;
  margin: 12px 0;
}

.skeleton .skeleton-line.short {
  width: 60%;
}

.skeleton .skeleton-line.xsmall {
  width: 30%;
  height: 12px;
}

.skeleton .skeleton-price {
  width: 110px;
  height: 24px;
  background: linear-gradient(90deg, #f0f0f0, #e0e0e0, #f0f0f0);
  background-size: 200% 100%;
  animation: shimmer 1.5s linear infinite;
}

.skeleton .skeleton-btn {
  width: 110px;
  height: 40px;
  background: linear-gradient(90deg, #f0f0f0, #e0e0e0, #f0f0f0);
  background-size: 200% 100%;
  animation: shimmer 1.5s linear infinite;
}

.skeleton .skeleton-btn.small {
  width: 86px;
}

@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}

/* small screens */
@media (max-width: 768px) {
  .product-card {
    flex-direction: column;
    width: 90vw;
    max-width: 90vw;
  }

  .card-media {
    width: 100%;
    min-width: 100%;
    height: 300px;
    min-height: 300px;
    border-right: none;
    border-bottom: 3px solid var(--light-blue-bg);
  }

  .card-body {
    padding: 24px;
  }

  .price-row {
    flex-direction: column;
    gap: 16px;
    align-items: stretch;
  }

  .actions {
    flex-direction: column;
    width: 100%;
  }

  .btn {
    width: 100%;
  }

  .loader-wrap {
    min-height: 300px;
  }

  .unavailable-state {
    width: 90vw;
    max-width: 90vw;
    padding: 20px;
  }
}

.index-indicator {
  text-align: center;
  margin-top: 14px;
  color: var(--medium-gray);
  font-size: 0.9rem;
}
</style>