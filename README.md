# E-Commerce Catalog

Product catalog viewer yang menampilkan produk clothing dari FakeStore API dengan desain modern dan interaktif.

## 🎯 Fitur

- ✅ Fetch produk dari [FakeStore API](https://fakestoreapi.com/)
- ✅ Filter otomatis: hanya menampilkan **men's clothing** dan **women's clothing**
- ✅ Navigasi produk dengan tombol "Next product"
- ✅ Desain berbeda untuk kategori men's dan women's
- ✅ Skeleton loader saat fetch data
- ✅ Caching produk untuk performa lebih baik
- ✅ Prefetch produk berikutnya
- ✅ Gradient background yang menarik
- ✅ Responsive design

## 🛠️ Teknologi

- **Vue 3** (Composition API)
- **Vite** (Build tool)
- **Vanilla CSS** (No framework)
- **FakeStore API**

## 📦 Instalasi

```bash
# Clone repository
git clone https://github.com/[username]/ecommerce-catalog.git

# Masuk ke folder project
cd ecommerce-catalog

# Install dependencies
npm install

# Jalankan dev server
npm run dev
```

## 🎨 Design System

### Color Palette

- **Primary Blue**: `#002772`
- **Light Purple**: `#FDE2FF`
- **Dark Gray**: `#1E1E1E`
- **Magenta**: `#720060`
- **Light Blue**: `#D6E6FF`
- **Medium Gray**: `#3F3F3F`
- **Light Gray**: `#DCDCDC`
- **White**: `#FFFFFF`

### Komponen Desain

1. **Page-Men**: Desain untuk produk men's clothing (border & accent biru)
2. **Page-Women**: Desain untuk produk women's clothing (border & accent magenta)

## 📱 Struktur Project

```
ecommerce-catalog/
├── src/
│   ├── components/
│   │   └── ProductViewer.vue    # Komponen utama
│   ├── assets/
│   │   ├── base.css             # CSS variables & global styles
│   │   └── main.css
│   ├── App.vue                  # Root component
│   └── main.js
├── public/
├── package.json
└── vite.config.js
```

