<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal-content">

      <div class="modal-header">
        <div class="modal-icon-wrap">✨</div>
        <div>
          <h2 class="modal-title">Nuevo Producto</h2>
          <p class="modal-subtitle">Completa los campos para agregar al menú</p>
        </div>
        <button class="close-btn" @click="$emit('close')">✕</button>
      </div>

      <form @submit.prevent="submitForm" class="modal-form">

        <div class="form-group">
          <label>Nombre del plato</label>
          <input
            type="text"
            v-model="product.name"
            :class="{ 'input-error': errors.name }"
            placeholder="Ej. Lomo saltado"
          />
          <span v-if="errors.name" class="error-msg">{{ errors.name }}</span>
        </div>

        <div class="form-group">
          <label>Descripción</label>
          <textarea
            v-model="product.description"
            :class="{ 'input-error': errors.description }"
            placeholder="Una breve descripción del platillo..."
            rows="3"
          ></textarea>
          <span v-if="errors.description" class="error-msg">{{ errors.description }}</span>
        </div>

        <div class="form-group">
          <label>URL de la foto del plato</label>
          <input
            type="url"
            v-model="product.image"
            placeholder="Ej. https://enlace.com/foto-plato.jpg"
          />
        </div>

        <div class="form-row">
          <div class="form-group">
            <label>Precio</label>
            <div class="input-prefix-wrap">
              <span class="input-prefix">$</span>
              <input
                type="number"
                step="0.01"
                v-model="product.price"
                :class="{ 'input-error': errors.price }"
                placeholder="0.00"
                class="has-prefix"
              />
            </div>
            <span v-if="errors.price" class="error-msg">{{ errors.price }}</span>
          </div>
          <div class="form-group">
            <label>Stock</label>
            <input
              type="number"
              v-model="product.stock"
              :class="{ 'input-error': errors.stock }"
              placeholder="0"
            />
            <span v-if="errors.stock" class="error-msg">{{ errors.stock }}</span>
          </div>
        </div>

        <div class="form-group">
          <label>Categoría</label>
          <div class="category-options">
            <label
              v-for="cat in categories"
              :key="cat.value"
              class="category-option"
              :class="{ active: product.category === cat.value }"
            >
              <input
                type="radio"
                :value="cat.value"
                v-model="product.category"
                hidden
              />
              <span class="cat-emoji">{{ cat.emoji }}</span>
              <span class="cat-label">{{ cat.value }}</span>
            </label>
          </div>
        </div>

        <div class="modal-actions">
          <button type="button" class="btn-cancel" @click="$emit('close')">Cancelar</button>
          <button type="submit" class="btn-save">Guardar producto</button>
        </div>

      </form>
    </div>
  </div>
</template>

<script setup>
import { reactive } from 'vue'

const emit = defineEmits(['close', 'add'])

const categories = [
  { value: 'Plato Fuerte', emoji: '🍽️' },
  { value: 'Entrada',      emoji: '🥗' },
  { value: 'Bebida',       emoji: '🥤' },
  { value: 'Postre',       emoji: '🍮' },
]

// Agregado 'image' con valor inicial vacío
const product = reactive({ name: '', description: '', image: '', price: null, stock: null, category: 'Plato Fuerte' })
const errors  = reactive({ name: '', description: '', price: '', stock: '' })

const validate = () => {
  let isValid = true
  if (!product.name.trim())                       { errors.name        = 'El nombre es obligatorio'; isValid = false } else { errors.name        = '' }
  if (!product.description.trim())                { errors.description = 'La descripción es obligatoria'; isValid = false } else { errors.description = '' }
  if (!product.price || product.price <= 0)      { errors.price       = 'Debe ser mayor a 0'; isValid = false }       else { errors.price       = '' }
  if (product.stock === null || product.stock < 0) { errors.stock     = 'Stock inválido'; isValid = false }           else { errors.stock       = '' }
  return isValid
}

const submitForm = () => {
  if (validate()) {
    // Se incluye la URL de la imagen al emitir el evento hacia el componente padre
    emit('add', { 
      ...product, 
      price: parseFloat(product.price), 
      stock: parseInt(product.stock) 
    })
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=Playfair+Display:wght@500&display=swap');

/* ─── Overlay ────────────────────────────────────────────── */
.modal-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(26, 23, 20, 0.52);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
  animation: overlayIn 0.2s ease;
}

/* ─── Modal ──────────────────────────────────────────────── */
.modal-content {
  background: #ffffff;
  border-radius: 20px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 24px 48px rgba(26,23,20,0.14), 0 0 0 1px rgba(26,23,20,0.05);
  animation: boxIn 0.25s cubic-bezier(0.34, 1.4, 0.64, 1);
  font-family: 'DM Sans', sans-serif;
  overflow: hidden;
}

/* ─── Header ─────────────────────────────────────────────── */
.modal-header {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 24px 24px 20px;
  border-bottom: 1px solid #f0ece6;
}

.modal-icon-wrap {
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background: #faf8f5;
  border: 1px solid #e8e4de;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  flex-shrink: 0;
}

.modal-title {
  margin: 0 0 2px;
  font-family: 'Playfair Display', serif;
  font-size: 1.2rem;
  font-weight: 500;
  color: #1a1714;
  line-height: 1.2;
}

.modal-subtitle {
  margin: 0;
  font-size: 12px;
  color: #b0aca6;
  font-weight: 400;
}

.close-btn {
  margin-left: auto;
  background: transparent;
  border: 1px solid #e8e4de;
  border-radius: 8px;
  width: 32px;
  height: 32px;
  cursor: pointer;
  font-size: 13px;
  color: #b0aca6;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.18s;
  flex-shrink: 0;
}

.close-btn:hover {
  background: #faf8f5;
  border-color: #6b6560;
  color: #1a1714;
}

/* ─── Form ───────────────────────────────────────────────── */
.modal-form {
  padding: 20px 24px 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-row {
  display: flex;
  gap: 14px;
}

.form-row .form-group { flex: 1; }

label {
  font-size: 12px;
  font-weight: 600;
  color: #6b6560;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

input,
textarea,
select {
  padding: 10px 13px;
  border: 1px solid #e8e4de;
  border-radius: 10px;
  font-family: 'DM Sans', sans-serif;
  font-size: 14px;
  color: #1a1714;
  background: #faf8f5;
  transition: border-color 0.18s, box-shadow 0.18s, background 0.18s;
  outline: none;
  width: 100%;
  box-sizing: border-box;
}

input::placeholder,
textarea::placeholder { color: #c8c3bc; }

input:focus,
textarea:focus,
select:focus {
  border-color: #1a1714;
  background: #ffffff;
  box-shadow: 0 0 0 3px rgba(26,23,20,0.06);
}

textarea { resize: none; line-height: 1.55; }

/* Input con prefijo $ */
.input-prefix-wrap {
  position: relative;
}

.input-prefix {
  position: absolute;
  left: 13px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 14px;
  color: #b0aca6;
  font-weight: 500;
  pointer-events: none;
}

input.has-prefix { padding-left: 26px; }

/* Error state */
.input-error {
  border-color: #c8572a !important;
  background: #fdf0ee !important;
}

.error-msg {
  font-size: 11px;
  color: #c8572a;
  font-weight: 500;
  margin-top: 0;
}

/* ─── Categoría (radio buttons visuales) ─────────────────── */
.category-options {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

.category-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  padding: 10px 6px;
  border-radius: 10px;
  border: 1px solid #e8e4de;
  background: #faf8f5;
  cursor: pointer;
  transition: all 0.18s;
  text-transform: none;
  letter-spacing: 0;
  font-weight: 400;
  color: #6b6560;
}

.category-option:hover {
  border-color: #b0aca6;
  background: #f5f2ee;
}

.category-option.active {
  border-color: #1a1714;
  background: #1a1714;
  color: #ffffff;
}

.cat-emoji { font-size: 20px; line-height: 1; }
.cat-label { font-size: 11px; font-weight: 500; text-align: center; }

/* ─── Acciones ───────────────────────────────────────────── */
.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding-top: 4px;
  border-top: 1px solid #f0ece6;
  margin-top: 4px;
}

.btn-cancel {
  background: transparent;
  border: 1px solid #e8e4de;
  padding: 10px 20px;
  border-radius: 10px;
  font-family: 'DM Sans', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: #6b6560;
  cursor: pointer;
  transition: all 0.18s;
}

.btn-cancel:hover {
  background: #faf8f5;
  border-color: #b0aca6;
  color: #1a1714;
}

.btn-save {
  background: #2d6a4f;
  color: #ffffff;
  border: none;
  padding: 10px 22px;
  border-radius: 10px;
  font-family: 'DM Sans', sans-serif;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.18s, transform 0.1s;
}

.btn-save:hover  { background: #1e4d39; }
.btn-save:active { transform: scale(0.97); }

/* ─── Animaciones ────────────────────────────────────────── */
@keyframes overlayIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes boxIn {
  from { opacity: 0; transform: scale(0.9) translateY(16px); }
  to   { opacity: 1; transform: scale(1) translateY(0); }
}
</style>