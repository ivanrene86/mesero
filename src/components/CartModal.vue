<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal-content cart-modal">

      <div class="modal-header">
        <div class="modal-icon-wrap">🛒</div>
        <div>
          <h2 class="modal-title">Tu Carrito</h2>
          <p class="modal-subtitle">Mesa #5 · Carlos G.</p>
        </div>
        <button class="close-btn" @click="$emit('close')">✕</button>
      </div>

      <div v-if="cart.length === 0" class="empty-cart">
        <span class="empty-icon">🍽️</span>
        <p class="empty-title">El carrito está vacío</p>
        <p class="empty-hint">Agrega productos desde el menú</p>
      </div>

      <div v-else class="cart-body">
        <ul class="cart-list">
          <li v-for="item in cart" :key="item.id" class="cart-item">
            <div class="item-emoji">{{ categoryEmoji(item.category) }}</div>
            <div class="item-info">
              <strong class="item-name">{{ item.name }}</strong>
              <span class="item-meta">{{ item.quantity }} × ${{ item.price.toLocaleString('es-CO') }}</span>
            </div>
            <div class="item-actions">
              <span class="item-total">${{ (item.price * item.quantity).toLocaleString('es-CO') }}</span>
              <button class="btn-remove" @click="$emit('remove', item.id)" title="Quitar del carrito">
                ✕
              </button>
            </div>
          </li>
        </ul>

        <div class="cart-total-row">
          <span class="total-label">Total a pagar</span>
          <span class="total-amount">${{ total.toLocaleString('es-CO') }}</span>
        </div>

        <button class="btn-checkout" @click="generateInvoice">
          <span class="checkout-icon">💳</span>
          Generar Factura y Descargar
        </button>
      </div>

      <button class="btn-close-modal" @click="$emit('close')">Cerrar</button>

    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import html2pdf from 'html2pdf.js'

const props = defineProps({ cart: Array })
const emit  = defineEmits(['close', 'remove', 'checkout'])

const total = computed(() => props.cart.reduce((sum, item) => sum + (item.price * item.quantity), 0))

const currentDate = new Date().toLocaleDateString('es-ES', {
  year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit'
})

const emojiMap = {
  'Plato Fuerte': '🍽️',
  'Entrada':      '🥗',
  'Bebida':       '🥤',
  'Postre':       '🍮',
}
const categoryEmoji = (cat) => emojiMap[cat] ?? '🍽️'

const generateInvoice = () => {
  const invoiceHtml = `
    <div style="font-family: 'Helvetica', 'Arial', sans-serif; color: #333; padding: 40px; max-width: 800px; margin: auto; border: 1px solid #eee;">
      
      <div style="text-align: center; border-bottom: 3px solid #3b82f6; padding-bottom: 20px; margin-bottom: 30px;">
        <h1 style="margin: 0; color: #3b82f6; font-size: 32px;">🍔 Restaurante Vue</h1>
        <p style="margin: 5px 0 0 0; font-size: 16px; color: #666;">Factura de Venta</p>
      </div>

      <div style="margin-bottom: 30px; font-size: 15px; line-height: 1.6;">
        <p style="margin:0;"><strong>Fecha:</strong> ${currentDate}</p>
        <p style="margin:0;"><strong>Mesero:</strong> Carlos G.</p>
        <p style="margin:0;"><strong>Mesa:</strong> #5</p>
      </div>

      <table style="width: 100%; border-collapse: collapse; margin-bottom: 30px; font-size: 15px;">
        <thead>
          <tr style="background-color: #f8f9fa;">
            <th style="border: 1px solid #dee2e6; padding: 12px; text-align: left;">Producto</th>
            <th style="border: 1px solid #dee2e6; padding: 12px; text-align: center;">Cant.</th>
            <th style="border: 1px solid #dee2e6; padding: 12px; text-align: right;">P. Unit.</th>
            <th style="border: 1px solid #dee2e6; padding: 12px; text-align: right;">Total</th>
          </tr>
        </thead>
        <tbody>
          ${props.cart.map(item => `
            <tr>
              <td style="border: 1px solid #dee2e6; padding: 12px;">${item.name}</td>
              <td style="border: 1px solid #dee2e6; padding: 12px; text-align: center;">${item.quantity}</td>
              <td style="border: 1px solid #dee2e6; padding: 12px; text-align: right;">$${item.price.toLocaleString('es-CO')}</td>
              <td style="border: 1px solid #dee2e6; padding: 12px; text-align: right; font-weight: bold;">$${(item.price * item.quantity).toLocaleString('es-CO')}</td>
            </tr>
          `).join('')}
        </tbody>
      </table>

      <div style="text-align: right; border-top: 3px solid #333; padding-top: 20px; margin-top: 20px;">
        <h2 style="margin: 0 0 10px 0; color: #10b981; font-size: 28px;">TOTAL: $${total.value.toLocaleString('es-CO')}</h2>
        <p style="margin: 0; color: #666; font-size: 18px;">¡Gracias por su visita!</p>
      </div>

    </div>
  `

  const opt = {
    margin:      10,
    filename:    `factura_restaurante_${Date.now()}.pdf`,
    image:       { type: 'jpeg', quality: 0.98 },
    html2canvas: { scale: 2, useCORS: true },
    jsPDF:       { unit: 'mm', format: 'a4', orientation: 'portrait' }
  }

  html2pdf().from(invoiceHtml).set(opt).save()
    .then(() => { emit('checkout') })
    .catch(err => {
      console.error('Error al generar el PDF:', err)
      alert('Hubo un error al generar la factura.')
    })
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
  align-items: flex-start;
  padding-top: 48px;
  z-index: 100;
  animation: overlayIn 0.2s ease;
}

/* ─── Modal ──────────────────────────────────────────────── */
.modal-content {
  background: #ffffff;
  border-radius: 20px;
  width: 90%;
  max-width: 560px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 24px 48px rgba(26,23,20,0.14), 0 0 0 1px rgba(26,23,20,0.05);
  animation: boxIn 0.25s cubic-bezier(0.34, 1.4, 0.64, 1);
  font-family: 'DM Sans', sans-serif;
  display: flex;
  flex-direction: column;
}

.modal-content::-webkit-scrollbar { width: 4px; }
.modal-content::-webkit-scrollbar-track { background: transparent; }
.modal-content::-webkit-scrollbar-thumb { background: #e8e4de; border-radius: 4px; }

/* ─── Header ─────────────────────────────────────────────── */
.modal-header {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 24px 24px 20px;
  border-bottom: 1px solid #f0ece6;
  position: sticky;
  top: 0;
  background: #ffffff;
  z-index: 2;
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

/* ─── Vacío ──────────────────────────────────────────────── */
.empty-cart {
  padding: 52px 24px;
  text-align: center;
  color: #b0aca6;
}

.empty-icon {
  display: block;
  font-size: 2.8rem;
  margin-bottom: 14px;
  opacity: 0.5;
}

.empty-title {
  margin: 0 0 6px;
  font-size: 1rem;
  font-weight: 500;
  color: #6b6560;
}

.empty-hint {
  margin: 0;
  font-size: 13px;
  color: #b0aca6;
}

/* ─── Body ───────────────────────────────────────────────── */
.cart-body {
  padding: 16px 24px 24px;
  display: flex;
  flex-direction: column;
  gap: 0;
}

/* ─── Lista ──────────────────────────────────────────────── */
.cart-list {
  list-style: none;
  padding: 0;
  margin: 0 0 4px;
}

.cart-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 13px 0;
  border-bottom: 1px solid #f0ece6;
}

.cart-item:last-child { border-bottom: none; }

.item-emoji {
  width: 38px;
  height: 38px;
  border-radius: 10px;
  background: #faf8f5;
  border: 1px solid #e8e4de;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  flex-shrink: 0;
}

.item-info {
  display: flex;
  flex-direction: column;
  gap: 3px;
  flex-grow: 1;
  min-width: 0;
}

.item-name {
  font-size: 14px;
  font-weight: 600;
  color: #1a1714;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.item-meta {
  font-size: 12px;
  color: #b0aca6;
  font-weight: 400;
}

.item-actions {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-shrink: 0;
}

.item-total {
  font-size: 14px;
  font-weight: 700;
  color: #1a1714;
  letter-spacing: -0.02em;
  min-width: 52px;
  text-align: right;
}

.btn-remove {
  background: transparent;
  border: 1px solid #e8e4de;
  border-radius: 7px;
  width: 28px;
  height: 28px;
  cursor: pointer;
  font-size: 11px;
  color: #b0aca6;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.18s;
  flex-shrink: 0;
}

.btn-remove:hover {
  background: #fdf0ee;
  border-color: #c8572a;
  color: #c8572a;
}

/* ─── Total ──────────────────────────────────────────────── */
.cart-total-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 0 20px;
  border-top: 1px solid #e8e4de;
  margin-top: 4px;
}

.total-label {
  font-size: 13px;
  font-weight: 500;
  color: #6b6560;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

.total-amount {
  font-family: 'Playfair Display', serif;
  font-size: 1.6rem;
  font-weight: 500;
  color: #1a1714;
  letter-spacing: -0.02em;
}

/* ─── Checkout ───────────────────────────────────────────── */
.btn-checkout {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 14px;
  background: #1a1714;
  color: #ffffff;
  border: none;
  border-radius: 12px;
  font-family: 'DM Sans', sans-serif;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.18s, transform 0.1s;
  letter-spacing: 0.01em;
}

.btn-checkout:hover  { background: #333; }
.btn-checkout:active { transform: scale(0.98); }

.checkout-icon { font-size: 17px; }

/* ─── Cerrar ─────────────────────────────────────────────── */
.btn-close-modal {
  width: calc(100% - 48px);
  margin: 0 24px 24px;
  padding: 11px;
  background: transparent;
  border: 1px solid #e8e4de;
  border-radius: 10px;
  font-family: 'DM Sans', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: #6b6560;
  cursor: pointer;
  transition: all 0.18s;
}

.btn-close-modal:hover {
  background: #faf8f5;
  border-color: #b0aca6;
  color: #1a1714;
}

/* ─── Animaciones ────────────────────────────────────────── */
@keyframes overlayIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes boxIn {
  from { opacity: 0; transform: scale(0.92) translateY(14px); }
  to   { opacity: 1; transform: scale(1) translateY(0); }
}

@media (max-width: 480px) {
  .modal-overlay { padding-top: 20px; }
  .modal-header { padding: 16px 16px 12px; }
  .modal-content { max-height: 90vh; }
  .cart-body { padding: 12px 16px 16px; }
  .cart-item { gap: 8px; padding: 10px 0; }
  .item-name { font-size: 13px; max-width: 140px; }
  .item-total { font-size: 13px; min-width: 45px; }
  .total-amount { font-size: 1.35rem; }
  .btn-checkout { padding: 12px; font-size: 14px; }
  .btn-close-modal { width: calc(100% - 32px); margin: 0 16px 16px; padding: 10px; }
}
</style>