<template>
  <div class="app-container">
    
    <header class="main-header">
      <div class="header-brand">
        <h1 class="restaurant-name">El Zaguán</h1>
        <p class="restaurant-slogan">Sabores de nuestra tierra</p>
      </div>

      <div class="search-container-fluid">
        <span class="search-icon">🔍</span>
        <input 
          type="text" 
          v-model="searchQuery" 
          placeholder="Buscar platillo o bebida..." 
          class="search-input-new"
        />
      </div>

      <div class="header-actions">
        <button class="btn-new-product" @click="isAddModalOpen = true">
          ✨ Nuevo Producto
        </button>
        
        <button class="btn-cart-new" @click="isCartModalOpen = true">
          🛒 {{ cartItemsCount }} ítems • <span class="cart-total-price">{{ formatearCOP(cartTotal) }}</span>
        </button>
      </div>
    </header>

    <nav class="categories-nav">
      <button 
        v-for="cat in ['Todas', 'Plato Fuerte', 'Entrada', 'Bebida', 'Postre']" 
        :key="cat"
        :class="['filter-btn', { 'filter-btn--active': activeCategory === cat }]"
        @click="activeCategory = cat"
      >
        {{ cat }}
      </button>
    </nav>

    <main class="products-grid">
      <MenuItem 
        v-for="product in filteredProducts" 
        :key="product.id" 
        :item="product"
        @add-to-cart="addToCart"
        @delete-item="deleteProduct"
        @restock-item="restockProduct"
      />
    </main>

    <AddProductModal 
      v-if="isAddModalOpen" 
      @close="isAddModalOpen = false" 
      @add="addProduct" 
    />
    
    <CartModal 
      v-if="isCartModalOpen" 
      :cart="cart"
      @close="isCartModalOpen = false" 
      @remove="removeFromCart"
      @checkout="handleCheckout"
    />

    <CustomDialog 
      v-if="isDialogOpen"
      :config="dialogConfig"
      @close="isDialogOpen = false"
    />

  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import Swal from 'sweetalert2'
import MenuItem from './MenuItem.vue'
import AddProductModal from './AddProductModal.vue'
import CartModal from './CartModal.vue'
import CustomDialog from './CustomDialog.vue'

// --- ESTADOS LOCALES ---
const activeCategory = ref('Todas')
const searchQuery = ref('')
const isAddModalOpen = ref(false)
const isCartModalOpen = ref(false)
const isDialogOpen = ref(false)
const dialogConfig = ref({ title: '', message: '', type: 'info' })

// --- CONFIGURACIÓN DE NOTIFICACIONES FLOTANTES (TOAST) ---
const toast = Swal.mixin({
  toast: true,
  position: 'top-end',
  showConfirmButton: false,
  timer: 2500,
  timerProgressBar: true,
  didOpen: (toast) => {
    toast.onmouseenter = Swal.stopTimer
    toast.onmouseleave = Swal.resumeTimer
  }
})

// --- CARGAR DE STORAGE GENÉRICO ---
const loadFromStorage = (key, fallback) => {
  const saved = localStorage.getItem(key)
  if (!saved) return fallback
  
  const parsed = JSON.parse(saved)
  if (key === 'vue_menu' && Array.isArray(parsed) && parsed.length < fallback.length) {
    localStorage.removeItem(key)
    return fallback
  }
  return parsed
}

// --- MENÚ GASTRONÓMICO DE 30 PRODUCTOS COLOMBIANOS REALES ---
const defaultMenu = [
  // Platos Fuertes
  { id: 1, name: 'Bandeja Paisa', description: 'Tradicional con frijoles, arroz, carne molida, chicharrón, huevo frito, tajada de plátano, chorizo y arepa.', price: 34000, category: 'Plato Fuerte', stock: 12, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQyTIp3Rk2PeoS2HO0otMJCmGAis4tgB0LGyw&s' },
  { id: 2, name: 'Ajiaco Santafereño', description: 'Sopa típica de Bogotá elaborada con tres tipos de papa, pollo desmechado, mazorca, acompañada de alcaparras y crema de leche.', price: 28000, category: 'Plato Fuerte', stock: 8, image: 'https://www.recetasnestle.com.co/sites/default/files/srh_recipes/f78cf6630b31638994b09b3b470b085c.jpg' },
  { id: 3, name: 'Sancocho de Gallina', description: 'Sopa espesa cocida a la leña con gallina campesina, plátano verde, yuca, papa y mazorca. Incluye arroz y aguacate.', price: 26000, category: 'Plato Fuerte', stock: 10, image: 'https://www.infobae.com/resizer/v2/R4VENSPXO5DZBJ7M5CNWTKEAJQ.jpg?auth=e17bf2694a3eaf395d58a160b09cac133ff395e55f3e48295ed3c66173c5af23&smart=true&width=350&height=196&quality=85' },
  { id: 4, name: 'Sobrebarriga en Salsa', description: 'Corte de carne tierno bañado en un tradicional ahogado criollo de tomate y cebolla, acompañado de papa y yuca chorreada.', price: 31000, category: 'Plato Fuerte', stock: 6, image: 'https://img-global.cpcdn.com/recipes/acb5472438a295fb/1200x630cq80/photo.jpg' },
  { id: 5, name: 'Lechona Tolimense', description: 'Delicioso cerdo relleno de arroz, arvejas amarillas y carne sazonada, cocido al horno hasta dorar el cuero.', price: 18000, category: 'Plato Fuerte', stock: 15, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQXxXTzW4gbv3Xqad1eS0M-YzcXuk6zoseI1A&s' },
  { id: 6, name: 'Arroz con Pollo Criollo', description: 'Clásico arroz desmechado con verduras selectas, raíces finas, servido con papas a la francesa crujientes y salsa de tomate.', price: 22000, category: 'Plato Fuerte', stock: 14, image: 'https://www.recetasnestle.com.ve/sites/default/files/styles/recipe_detail_desktop_new/public/srh_recipes/abca4d5b6e6c7e97ece98ed0faaeb211.png?itok=tXIPmPYV' },
  { id: 7, name: 'Mote de Queso', description: 'Sopa cremosa costeña a base de ñame crudo, queso costeño picado, suero costeño y un refrito de ajo y cebolla.', price: 24000, category: 'Plato Fuerte', stock: 7, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRvHulOV3ERhCMkLsadQiUcbzwWpu9q1rQuGg&s' },
  { id: 8, name: 'Pescado Frito Mojarra', description: 'Mojarra entera frita al punto crocante, acompañada de arroz de coco caribeño, patacones planos y ensalada fresca.', price: 32000, category: 'Plato Fuerte', stock: 9, image: 'https://recetas.encolombia.com/wp-content/uploads/2013/03/mojarra-frita.webp' },
  // Entradas
  { id: 9, name: 'Empanadas de Carne (3 unidades)', description: 'Crujientes empanadas de masa de maíz frito rellenas de carne y papa picada, acompañadas de ají casero picante.', price: 6000, category: 'Entrada', stock: 40, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQq1b3xdEnw-vy92rBIzFCENHmKBbpUaz9D6Q&s' },
  { id: 10, name: 'Arepa de Huevo Costeña', description: 'Arepa de maíz amarillo frita con un huevo entero en su interior, dorada a la perfección caribeña.', price: 5500, category: 'Entrada', stock: 20, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRwHhaSVtKfgYYem2VNn7JXUr9n3sx0C-KfGg&s' },
  { id: 11, name: 'Papas Criollas Fritas', description: 'Porción de papitas amarillas nativas fritas, espolvoreadas con sal marina fina y servidas con salsa rosada.', price: 8000, category: 'Entrada', stock: 25, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcROReyO_bvZ1Gpzz8I5RN23-6vdSzIzfJnouQ&s' },
  { id: 12, name: 'Patacones con Hogao', description: 'Cuatro patacones de plátano verde fritos andinos, cubiertos con un guiso generoso de tomate, cebolla y ajo.', price: 9500, category: 'Entrada', stock: 30, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRuI-s3FCv3_wZ5w4j6Z3TsWngAylEup_8sXA&s' },
  { id: 13, name: 'Carimañolas de Queso (3 unidades)', description: 'Pasteles fritos de masa de yuca suave rellenos de queso costeño derretido, deliciosamente crujientes.', price: 7500, category: 'Entrada', stock: 18, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQpNYf0y55dHPK03iLVhTm07i-DgdkUAvv4rQ&s' },
  { id: 14, name: 'Chunchulla Frita', description: 'Intestino delgado de res picado y frito hasta quedar crocante, sazonado con limón verde fresco y sal.', price: 12000, category: 'Entrada', stock: 11, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRk4ngDNUn1ktMjgfRR1oraGC98QQxrAKAA_Q&s' },
  { id: 15, name: 'Arepa Santandereana', description: 'Arepa de maíz pelado cocido con chicharrón de cerdo molido, asada lentamente en tiesto de barro.', price: 4000, category: 'Entrada', stock: 35, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQr9drEVckd4CH4Ir1KumbaXO9iBmV6ruZs9A&s' },
  { id: 16, name: 'Chorizo con Arepa', description: 'Chorizo parrillero artesanal de cerdo ahumado, servido caliente con una arepita blanca paisa asada.', price: 8500, category: 'Entrada', stock: 22, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRu81uCUaHxNiu0cul9FFXXVUgYNf2HD0ovIw&s' },
  // Bebidas
  { id: 17, name: 'Limonada de Coco', description: 'Bebida refrescante granizada de zumo de limón natural mezclado con deliciosa crema de coco endulzada.', price: 9000, category: 'Bebida', stock: 30, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQheO01DBDn5eLsQcqQwZKwza-Aqx_kKZRQUw&s' },
  { id: 18, name: 'Jugo de Maracuyá en Agua', description: 'Jugo natural de la fruta de la pasión licuado al instante, refrescante, cítrico y dulce.', price: 6500, category: 'Bebida', stock: 45, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSd544_bSNqwa1pJx8rMYBWAmOIGJlkNIs5Kw&s' },
  { id: 19, name: 'Jugo de Lulo en Leche', description: 'Tradicional jugo de lulo (naranjilla) preparado en base láctea cremosa, espumoso y frío.', price: 7500, category: 'Bebida', stock: 40, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSSko8Y07T3ZHi8lXJrKxmpPJloQIau6LKIiw&s' },
  { id: 20, name: 'Agua de Panela con Queso', description: 'Infusión caliente de panela pura concentrada, servida con una buena tajada de queso fresco para derretir.', price: 6000, category: 'Bebida', stock: 15, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ4pyK4RyAka4BZBlK-mqDF_3Ki9cdg-T5HFw&s' },
  { id: 21, name: 'Refajo Colombiano', description: 'Mezcla perfecta y tradicional de cerveza nacional rubia fría con gaseosa Colombiana dulce de ampolla.', price: 8000, category: 'Bebida', stock: 50, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRyvESD3ZxcS7pb1Ok4rOsS6NKswMIt3txlIw&s' },
  { id: 22, name: 'Chocolate Completo', description: 'Taza de chocolate negro caliente en leche, acompañado de una arepa con mantequilla y queso campesino.', price: 9000, category: 'Bebida', stock: 12, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ4J4tVLYP3sZZ7ACqmBMYOYDC29xhhAMmnLQ&s' },
  { id: 23, name: 'Café Tinto Filtrado', description: 'Taza de café suave de origen colombiano, filtrado artesanalmente y endulzado opcionalmente con panela.', price: 3500, category: 'Bebida', stock: 60, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSOksD-ns_wUCOY8wA4TlQfp25BzJkLgrfIrQ&s' },
  // Postres
  { id: 24, name: 'Arequipe con Brevas', description: 'Brevas en almíbar de azúcar cortadas en cruz, rellenas de un suave y cremoso dulce de leche arequipe.', price: 8000, category: 'Postre', stock: 20, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQTeK_qxLoYbqp5zvRv5f6FZsV6k5Cz1G01Dg&s' },
  { id: 25, name: 'Cuajada con Melado', description: 'Porción generosa de cuajada de leche fresca bañada en un espeso y dulce almíbar caliente de panela pura.', price: 7500, category: 'Postre', stock: 14, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRTTpFS-bvcEqcZJlnvCxIifjaJ08tiNOf2Xg&s' },
  { id: 26, name: 'Arroz con Leche Casero', description: 'Postre cremoso de arroz cocido con leche condensada, canela aromática en astillas y pasas dulces.', price: 7000, category: 'Postre', stock: 22, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRzpYBcv3myDA3CC2SYbx2UMQbQjAwy-KmtSg&s' },
  { id: 27, name: 'Merengón de Fresas', description: 'Disco de merengue horneado crujiente, relleno de abundante crema batida montada y fresas frescas picadas.', price: 11000, category: 'Postre', stock: 10, image: 'https://www.recetasnestle.com.co/sites/default/files/srh_recipes/2b6358d0ce962d1ef68cba4ebcd1308b.jpg' },
  { id: 28, name: 'Postre de Natas', description: 'Dulce tradicional andino elaborado con las natas suaves de la leche hervida, huevo, almíbar y pasas.', price: 9500, category: 'Postre', stock: 8, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRs-2rr5irgi3Rv9_Sf_L3oalGUcRq7sqo1Jg&s' },
  { id: 29, name: 'Flan de Tres Leches', description: 'Flan horneado suave y de textura lisa, bañado en caramelo líquido dorado y un toque de vainilla.', price: 8500, category: 'Postre', stock: 16, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQePDb-AOsRE1ggcHlSbUdL8krwxJlz3wLQgg&s' },
  { id: 30, name: 'Enyucado Costeño', description: 'Torta tradicional de la costa atlántica a base de yuca rallada, coco fresco, queso, azúcar y semillas de anís.', price: 8000, category: 'Postre', stock: 12, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTPaFl7ksRf-udi-KhSzQ-h_Gch9LX2fz6FRg&s' }
]

// --- ASIGNACIÓN DE ESTADOS INICIALES ---
const menuItems = ref(loadFromStorage('vue_menu', defaultMenu))
const cart = ref(loadFromStorage('vue_cart', []))

// --- GUARDADO AUTOMÁTICO EN CACHÉ ---
watch(menuItems, (newMenu) => {
  localStorage.setItem('vue_menu', JSON.stringify(newMenu))
}, { deep: true })

watch(cart, (newCart) => {
  localStorage.setItem('vue_cart', JSON.stringify(newCart))
}, { deep: true })

// --- FILTRADO Y BÚSQUEDA ---
const filteredProducts = computed(() => {
  return menuItems.value.filter(item => {
    const matchesCategory = activeCategory.value === 'Todas' || item.category === activeCategory.value
    const matchesSearch = item.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
                          item.description.toLowerCase().includes(searchQuery.value.toLowerCase())
    return matchesCategory && matchesSearch
  })
})

const cartItemsCount = computed(() => cart.value.reduce((total, item) => total + item.quantity, 0))
const cartTotal = computed(() => cart.value.reduce((total, item) => total + (item.price * item.quantity), 0))

// --- MÉTODOS Y ACCIONES ---
const addToCart = (product) => {
  const targetProduct = menuItems.value.find(p => p.id === product.id)
  if (targetProduct && targetProduct.stock > 0) {
    targetProduct.stock--
    const cartItem = cart.value.find(item => item.id === product.id)
    if (cartItem) {
      cartItem.quantity++
    } else {
      cart.value.push({ ...product, quantity: 1 })
    }
    
    toast.fire({
      icon: 'success',
      title: `Agregado: ${product.name}`
    })
  } else {
    toast.fire({
      icon: 'error',
      title: '¡Platillo agotado de momento!'
    })
  }
}

const removeFromCart = (productId) => {
  const cartIndex = cart.value.findIndex(item => item.id === productId)
  if (cartIndex !== -1) {
    const name = cart.value[cartIndex].name
    const productInMenu = menuItems.value.find(p => p.id === productId)
    if (productInMenu) productInMenu.stock += cart.value[cartIndex].quantity
    cart.value.splice(cartIndex, 1)

    toast.fire({
      icon: 'info',
      title: `Removido: ${name}`
    })
  }
}

const addProduct = (newProd) => {
  const id = menuItems.value.length ? Math.max(...menuItems.value.map(p => p.id)) + 1 : 1
  
  // Respaldo dinámico por si el usuario introduce una URL con bloqueos CORS/Hotlinking externos
  const secureImage = newProd.image && newProd.image.trim() !== '' 
    ? newProd.image 
    : 'https://images.unsplash.com/photo-1504674900247-0877df9cc836?w=600&auto=format&fit=crop&q=80'

  menuItems.value.push({ 
    id, 
    ...newProd,
    image: secureImage,
    stock: parseInt(newProd.stock) || 0, 
    price: parseFloat(newProd.price) || 0 
  })
  isAddModalOpen.value = false

  Swal.fire({
    title: '¡Producto Registrado!',
    text: `"${newProd.name}" ha sido añadido con éxito al menú.`,
    icon: 'success',
    confirmButtonColor: '#1a1714'
  })
}

const deleteProduct = (id) => {
  const product = menuItems.value.find(p => p.id === id)
  const name = product ? product.name : 'este producto'

  Swal.fire({
    title: '¿Deseas eliminar este producto?',
    text: `Vas a borrar "${name}" de la lista de forma permanente.`,
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#d33',
    cancelButtonColor: '#b0aca6',
    confirmButtonText: 'Sí, borrarlo',
    cancelButtonText: 'Cancelar',
    reverseButtons: true
  }).then((result) => {
    if (result.isConfirmed) {
      cart.value = cart.value.filter(item => item.id !== id)
      menuItems.value = menuItems.value.filter(p => p.id !== id)
      
      Swal.fire({
        title: 'Eliminado',
        text: 'El producto ha sido removido del menú.',
        icon: 'success',
        confirmButtonColor: '#1a1714'
      })
    }
  })
}

const restockProduct = (id) => {
  const product = menuItems.value.find(p => p.id === id)
  if (product) {
    product.stock += 10
    toast.fire({
      icon: 'success',
      title: `+10 Unidades añadidas a ${product.name}`
    })
  }
}

const handleCheckout = () => {
  isCartModalOpen.value = false
  cart.value = []
  localStorage.removeItem('vue_cart')
  
  Swal.fire({
    title: '¡Pedido Exitoso!',
    text: 'La orden ha sido enviada directamente a la cocina de El Zaguán.',
    icon: 'success',
    confirmButtonColor: '#1a1714'
  })
}

const formatearCOP = (valor) => {
  return new Intl.NumberFormat('es-CO', { style: 'currency', currency: 'COP', minimumFractionDigits: 0 }).format(valor)
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=Playfair+Display:wght@600;700&display=swap');

.app-container {
  max-width: 1250px;
  margin: 0 auto;
  padding: 24px;
  background-color: #faf8f5;
  min-height: 100vh;
}

/* Rediseño del Header: Fuerza que todo viva en una sola línea horizontal */
.main-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #ffffff;
  padding: 20px 28px;
  border-radius: 16px;
  border: 1px solid #e8e4de;
  box-shadow: 0 2px 10px rgba(26,23,20,0.02);
  margin-bottom: 24px;
  gap: 20px;
  position: sticky;
  top: 0;
  z-index: 100;
  background: #ffffff;
}

.header-brand {
  flex-shrink: 0;
}
.restaurant-name {
  font-family: 'Playfair Display', serif;
  font-size: 1.8rem;
  font-weight: 700;
  color: #1a1714;
  margin: 0;
  line-height: 1.1;
}
.restaurant-slogan {
  font-family: 'DM Sans', sans-serif;
  font-size: 0.85rem;
  color: #b0aca6;
  margin: 4px 0 0 0;
  letter-spacing: 0.03em;
}

/* Buscador equilibrado en el centro horizontal */
.search-container-fluid {
  position: relative;
  flex: 1;
  max-width: 420px;
  margin: 0 15px;
}
.search-icon {
  position: absolute;
  left: 16px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 14px;
  color: #b0aca6;
}
.search-input-new {
  width: 100%;
  box-sizing: border-box; /* Previene desbordamiento */
  padding: 12px 16px 12px 46px;
  border-radius: 100px;
  border: 1px solid #e8e4de;
  background: #faf8f5;
  font-family: 'DM Sans', sans-serif;
  font-size: 0.9rem;
  color: #1a1714;
  transition: all 0.2s ease;
}
.search-input-new:focus {
  outline: none;
  border-color: #1a1714;
  background: #ffffff;
  box-shadow: 0 0 0 3px rgba(26,23,20,0.04);
}

/* Grupo de acciones alineadas a la derecha */
.header-actions {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-shrink: 0;
}

.btn-new-product {
  background: #ffffff;
  color: #6b6560;
  border: 1px solid #e8e4de;
  padding: 12px 20px;
  border-radius: 100px;
  font-family: 'DM Sans', sans-serif;
  font-size: 0.85rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  white-space: nowrap;
}
.btn-new-product:hover {
  background: #faf8f5;
  border-color: #6b6560;
}

/* Botón del Carrito: Con un ancho estricto e inmutable */
.btn-cart-new {
  background: #1a1714;
  color: #ffffff;
  border: none;
  padding: 12px 0; /* Controlamos el padding horizontal con un ancho fijo */
  width: 240px;    /* 🌟 ANCHO FIJO: No cambia de tamaño al meter productos */
  text-align: center;
  border-radius: 100px;
  font-family: 'DM Sans', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
  white-space: nowrap;
  display: inline-block;
}
.btn-cart-new:hover {
  background: #333333;
}
.cart-total-price {
  color: #fdf4e3;
}

.categories-nav {
  display: flex;
  gap: 10px;
  margin-bottom: 28px;
  overflow-x: auto;
  padding-bottom: 5px;
}
.filter-btn {
  background: #ffffff;
  border: 1px solid #e8e4de;
  padding: 9px 20px;
  border-radius: 100px;
  font-family: 'DM Sans', sans-serif;
  font-size: 0.85rem;
  color: #6b6560;
  cursor: pointer;
  transition: all 0.18s;
  white-space: nowrap;
}
.filter-btn:hover {
  background: #fdfbf7;
  border-color: #b0aca6;
}
.filter-btn--active {
  background: #1a1714;
  color: #ffffff;
  border-color: #1a1714;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 24px;
}
/* ==========================================================================
   🚨 SECCIÓN NUEVA: MEDIA QUERIES PARA RESPONSIVIDAD (SIN ALTERAR LO ANTERIOR)
   ========================================================================== */

/* 1. Tablets y Pantallas Medianas (max-width: 992px) */
@media (max-width: 992px) {
  .main-header {
    padding: 18px 20px;
    gap: 15px;
  }
  
  .restaurant-name {
    font-size: 1.6rem;
  }
  
  .search-container-fluid {
    max-width: 300px;
    margin: 0 5px;
  }
}

/* 2. Celulares y Dispositivos Móviles (max-width: 768px) */
@media (max-width: 768px) {
  .app-container {
    padding: 12px; /* Reducimos márgenes de la app para ganar pantalla */
  }

  /* Cambiamos el Header de fila a columna para acomodar los elementos */
  .main-header {
    flex-direction: column;
    align-items: stretch; /* Ocupa todo el ancho */
    padding: 16px;
    gap: 16px;
    text-align: center;
    position: sticky;
    top: 0;
    z-index: 100;
    background: #ffffff;
  }

  .header-brand {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  /* El buscador toma todo el ancho en móviles */
  .search-container-fluid {
    max-width: 100%;
    margin: 0;
  }

  /* Las acciones se reparten equitativamente abajo */
  .header-actions {
    width: 100%;
    justify-content: space-between;
    gap: 10px;
  }

  .btn-new-product {
    flex: 1;
    padding: 12px 10px;
    font-size: 0.8rem;
    text-align: center;
  }

  /* El botón de carrito pasa de ancho fijo a flexible en pantallas pequeñas */
  .btn-cart-new {
    flex: 1.3;
    width: auto; /* Rompe el ancho fijo solo en celulares para que no se salga */
    padding: 12px 10px;
    font-size: 0.8rem;
  }
  
  /* 🛠️ MODIFICADO: Cambiado a 1 columna fija si la pantalla ya es tamaño móvil estándar */
  .products-grid {
    grid-template-columns: 1fr; 
    gap: 16px;
  }
}

/* 3. Celulares Pequeños (max-width: 480px) */
@media (max-width: 480px) {
  .header-actions {
    flex-direction: column; /* Apila el botón nuevo y el carrito si la pantalla es enana */
  }
  
  .btn-new-product, .btn-cart-new {
    width: 100%;
    flex: none;
  }

  /* 🛠️ AÑADIDO: Forzamos una sola columna limpia y evitamos desbordes */
  .products-grid {
    grid-template-columns: 1fr !important;
    width: 100%;
    padding: 0;
  }

  /* Asegura que las tarjetas hijas se adapten al 100% real del ancho del teléfono */
  .products-grid > * {
    width: 100% !important;
    max-width: 100% !important;
    box-sizing: border-box;
  }
}
</style>