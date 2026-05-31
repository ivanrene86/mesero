<template>
  <div class="card" :class="{ 'out-of-stock': item.stock === 0 }">

    <button class="delete-btn" @click="$emit('delete-item', item.id)" title="Eliminar producto">
      🗑️
    </button>

    <div class="card-img" :class="`card-img--${categoryKey}`">
      <span class="category-badge">{{ item.category }}</span>
      
      <img 
        v-if="item.image" 
        :src="item.image" 
        :alt="item.name" 
        class="product-image"
        @error="(e) => e.target.style.display = 'none'" 
      />
      
      <span class="card-emoji">{{ categoryEmoji }}</span>
    </div>

    <div class="card-body">
      <h3 class="title">{{ item.name }}</h3>
      <p class="description">{{ item.description }}</p>

      <div class="stock-container">
        <div class="stock-info">
          <span class="stock-dot" :class="item.stock > 0 ? 'dot-green' : 'dot-red'"></span>
          <span class="stock-text">
            Stock: <strong :class="item.stock > 0 ? 'in-stock' : 'no-stock'">{{ item.stock }}</strong>
          </span>
        </div>
        <button class="restock-btn" @click="$emit('restock-item', item.id)" title="Reabastecer inventario">
          📦 +Stock
        </button>
      </div>

      <div class="card-footer">
        <span class="price">${{ item.price.toFixed(2) }}</span>
        <button
          class="add-btn"
          @click="$emit('add-to-cart', item)"
          :disabled="item.stock === 0"
        >
          {{ item.stock === 0 ? 'Agotado' : '+ Agregar' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  item: {
    type: Object,
    required: true
  }
})

defineEmits(['add-to-cart', 'delete-item', 'restock-item'])

const categoryMap = {
  'Plato Fuerte': { key: 'main',    emoji: '🍽️' },
  'Entrada':      { key: 'starter', emoji: '🥗' },
  'Bebida':       { key: 'drink',   emoji: '🥤' },
  'Postre':       { key: 'dessert', emoji: '🍮' },
}

const categoryKey   = computed(() => categoryMap[props.item.category]?.key   ?? 'main')
const categoryEmoji = computed(() => categoryMap[props.item.category]?.emoji ?? '🍽️')
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=Playfair+Display:wght@500&display=swap');

/* ─── Card ───────────────────────────────────────────────── */
.card {
  background: #ffffff;
  border-radius: 16px;
  overflow: visible;
  border: 1px solid #e8e4de;
  box-shadow: 0 1px 4px rgba(26,23,20,0.05);
  transition: transform 0.22s ease, box-shadow 0.22s ease;
  display: flex;
  flex-direction: column;
  position: relative;
  font-family: 'DM Sans', sans-serif;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 28px rgba(26,23,20,0.10);
}

.out-of-stock {
  opacity: 0.55;
}

/* ─── Botón Eliminar ─────────────────────────────────────── */
.delete-btn {
  position: absolute;
  top: 10px;
  left: 10px;
  background: rgba(255,255,255,0.92);
  border: 1px solid #e8e4de;
  border-radius: 50%;
  width: 32px;
  height: 32px;
  cursor: pointer;
  font-size: 14px;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.18s, border-color 0.18s, transform 0.1s;
  box-shadow: 0 1px 4px rgba(26,23,20,0.08);
}

.delete-btn:hover {
  background: #fdf0ee;
  border-color: #c8572a;
}

.delete-btn:active { transform: scale(0.92); }

/* ─── Header / Imagen ────────────────────────────────────── */
.card-img {
  height: 118px;
  border-radius: 15px 15px 0 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2.8rem;
  position: relative;
  overflow: hidden;
}

/* Nuevos estilos para la etiqueta img incorporada */
.product-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
}

/* Colores por categoría — cálidos y coherentes con la paleta crema */
.card-img--main    { background: #f5ede8; }
.card-img--starter { background: #e8f4ef; }
.card-img--drink   { background: #e8eef8; }
.card-img--dessert { background: #fdf4e3; }

.card-emoji {
  font-size: 2.6rem;
  line-height: 1;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.08));
  position: relative;
  z-index: 0; /* Queda por detrás de la imagen de comida */
}

/* ─── Badge de categoría ─────────────────────────────────── */
.category-badge {
  position: absolute;
  top: 10px;
  right: 10px;
  background: rgba(255,255,255,0.88);
  backdrop-filter: blur(4px);
  padding: 4px 10px;
  border-radius: 100px;
  font-size: 11px;
  font-weight: 600;
  color: #6b6560;
  letter-spacing: 0.03em;
  border: 1px solid rgba(232,228,222,0.9);
  z-index: 2; /* Siempre por encima de la imagen */
}

/* ─── Cuerpo ─────────────────────────────────────────────── */
.card-body {
  padding: 16px;
  display: flex;
  flex-direction: column;
  flex-grow: 1;
  gap: 0;
}

.title {
  margin: 0 0 6px;
  font-family: 'Playfair Display', serif;
  font-size: 1rem;
  font-weight: 500;
  color: #1a1714;
  line-height: 1.3;
}

.description {
  margin: 0 0 14px;
  font-size: 0.82rem;
  color: #b0aca6;
  line-height: 1.55;
  flex-grow: 1;
}

/* ─── Stock ──────────────────────────────────────────────── */
.stock-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 14px;
  background: #faf8f5;
  padding: 8px 10px;
  border-radius: 8px;
  border: 1px solid #f0ece6;
}

.stock-info {
  display: flex;
  align-items: center;
  gap: 7px;
}

.stock-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  flex-shrink: 0;
}

.dot-green { background: #2d6a4f; }
.dot-red   { background: #c8572a; }

.stock-text {
  font-size: 12px;
  color: #6b6560;
  font-weight: 400;
}

.stock-text strong { font-weight: 600; }
.in-stock { color: #2d6a4f; }
.no-stock { color: #c8572a; }

.restock-btn {
  background: #ffffff;
  border: 1px solid #e8e4de;
  border-radius: 6px;
  cursor: pointer;
  font-family: 'DM Sans', sans-serif;
  font-size: 11px;
  font-weight: 600;
  padding: 5px 10px;
  color: #6b6560;
  transition: all 0.18s;
  letter-spacing: 0.01em;
}

.restock-btn:hover {
  background: #1a1714;
  color: #ffffff;
  border-color: #1a1714;
}

/* ─── Footer ─────────────────────────────────────────────── */
.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid #f0ece6;
  padding-top: 13px;
}

.price {
  font-size: 1.15rem;
  font-weight: 700;
  color: #1a1714;
  letter-spacing: -0.02em;
}

.add-btn {
  background: #1a1714;
  color: #ffffff;
  border: none;
  padding: 8px 16px;
  border-radius: 8px;
  font-family: 'DM Sans', sans-serif;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.18s, transform 0.1s;
  letter-spacing: 0.01em;
}

.add-btn:hover:not(:disabled) {
  background: #333;
}

.add-btn:active:not(:disabled) { transform: scale(0.96); }

.add-btn:disabled {
  background: #e8e4de;
  color: #b0aca6;
  cursor: not-allowed;
}

/* ==========================================================================
   🚨 SECCIÓN NUEVA: MEDIA QUERIES PARA ADAPTAR LAS TARJETAS EN MÓVILES
   ========================================================================== */
@media (max-width: 480px) {
  .card-body {
    padding: 12px; /* Espaciado interno un poco más estrecho */
  }

  .title {
    font-size: 0.95rem; /* El título disminuye levemente para no ocupar tantas líneas */
  }

  .stock-container {
    padding: 6px 8px; /* Hace el módulo de stock un poco más delgado */
  }

  .restock-btn {
    padding: 4px 6px; /* Ajusta el tamaño del botón secundario */
    font-size: 10px;
  }

  .price {
    font-size: 1.05rem; /* Ajusta la escala del precio */
  }

  .add-btn {
    padding: 6px 12px; /* Compacta el botón de añadir para pantallas angostas */
    font-size: 12px;
  }
}
</style>