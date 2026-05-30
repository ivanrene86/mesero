<template>
  <div class="dialog-overlay" @click.self="cancel">
    <div class="dialog-box" :class="`dialog-box--${type}`">

      <!-- Ícono según tipo -->
      <div class="dialog-icon-wrap">
        <span class="dialog-icon">
          {{ type === 'alert' ? '⚠️' : type === 'prompt' ? '📦' : '🗑️' }}
        </span>
      </div>

      <h3 class="dialog-title">{{ title }}</h3>
      <p class="dialog-message">{{ message }}</p>

      <!-- Input solo aparece si es tipo 'prompt' (reabastecer) -->
      <div v-if="type === 'prompt'" class="dialog-input-group">
        <input
          type="number"
          v-model="inputValue"
          class="dialog-input"
          @keyup.enter="confirm"
          ref="inputRef"
          min="1"
        />
        <span class="input-hint">unidades</span>
      </div>

      <div class="dialog-actions">
        <!-- Botón de cancelar (no aparece en alertas simples) -->
        <button
          v-if="type !== 'alert'"
          class="btn-cancel"
          @click="cancel"
        >
          Cancelar
        </button>
        <button
          class="btn-confirm"
          :class="`btn-confirm--${type}`"
          @click="confirm"
        >
          {{ type === 'alert' ? 'Entendido' : 'Confirmar' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  type: { type: String, default: 'alert' }, // 'alert', 'confirm', 'prompt'
  title: { type: String, default: 'Notificación' },
  message: { type: String, default: '' },
  defaultValue: { type: String, default: '' }
})

const emit = defineEmits(['close', 'confirm'])

const inputValue = ref(props.defaultValue)
const inputRef = ref(null)

onMounted(() => {
  if (props.type === 'prompt' && inputRef.value) {
    inputRef.value.focus()
    inputRef.value.select()
  }
})

const confirm = () => {
  if (props.type === 'prompt') {
    const amount = parseInt(inputValue.value)
    if (isNaN(amount) || amount <= 0) {
      return
    }
    emit('confirm', amount)
  } else {
    emit('confirm', true)
  }
}

const cancel = () => {
  emit('close')
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=Playfair+Display:wght@500&display=swap');

/* ─── Overlay ────────────────────────────────────────────── */
.dialog-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(26, 23, 20, 0.55);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 200;
  animation: overlayIn 0.2s ease;
}

/* ─── Box ────────────────────────────────────────────────── */
.dialog-box {
  background: #ffffff;
  padding: 32px 32px 28px;
  border-radius: 20px;
  width: 90%;
  max-width: 400px;
  box-shadow: 0 24px 48px rgba(26, 23, 20, 0.16), 0 0 0 1px rgba(26,23,20,0.06);
  animation: boxIn 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
  text-align: center;
  font-family: 'DM Sans', sans-serif;
}

/* ─── Ícono ──────────────────────────────────────────────── */
.dialog-icon-wrap {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  background: #faf8f5;
  border: 1px solid #e8e4de;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 20px;
}

.dialog-icon {
  font-size: 22px;
  line-height: 1;
}

/* ─── Texto ──────────────────────────────────────────────── */
.dialog-title {
  margin: 0 0 10px;
  font-family: 'Playfair Display', serif;
  font-size: 1.25rem;
  font-weight: 500;
  color: #1a1714;
  line-height: 1.3;
}

.dialog-message {
  margin: 0 0 24px;
  color: #6b6560;
  font-size: 0.93rem;
  line-height: 1.65;
}

/* ─── Input ──────────────────────────────────────────────── */
.dialog-input-group {
  margin-bottom: 24px;
  display: flex;
  align-items: center;
  gap: 10px;
  justify-content: center;
}

.dialog-input {
  width: 110px;
  padding: 11px 14px;
  border: 1.5px solid #e8e4de;
  border-radius: 10px;
  font-family: 'DM Sans', sans-serif;
  font-size: 1.3rem;
  font-weight: 500;
  text-align: center;
  color: #1a1714;
  background: #faf8f5;
  box-sizing: border-box;
  transition: border-color 0.2s, box-shadow 0.2s;
  outline: none;
}

.dialog-input::-webkit-outer-spin-button,
.dialog-input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }

.dialog-input:focus {
  border-color: #1a1714;
  background: #ffffff;
  box-shadow: 0 0 0 3px rgba(26,23,20,0.06);
}

.input-hint {
  font-size: 13px;
  color: #b0aca6;
  font-weight: 400;
}

/* ─── Acciones ───────────────────────────────────────────── */
.dialog-actions {
  display: flex;
  justify-content: center;
  gap: 10px;
}

.btn-cancel {
  background: transparent;
  border: 1px solid #e8e4de;
  padding: 10px 22px;
  border-radius: 10px;
  font-family: 'DM Sans', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: #6b6560;
  cursor: pointer;
  transition: background 0.2s, border-color 0.2s, color 0.2s;
}

.btn-cancel:hover {
  background: #faf8f5;
  border-color: #b0aca6;
  color: #1a1714;
}

.btn-confirm {
  padding: 10px 22px;
  border-radius: 10px;
  border: none;
  font-family: 'DM Sans', sans-serif;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
  color: #ffffff;
}

.btn-confirm:active { transform: scale(0.97); }

/* Colores según tipo */
.btn-confirm--alert   { background: #c8572a; }
.btn-confirm--alert:hover { background: #a8441e; }

.btn-confirm--confirm { background: #1a1714; }
.btn-confirm--confirm:hover { background: #333; }

.btn-confirm--prompt  { background: #2d6a4f; }
.btn-confirm--prompt:hover { background: #1e4d39; }

/* ─── Animaciones ────────────────────────────────────────── */
@keyframes overlayIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes boxIn {
  from { opacity: 0; transform: scale(0.88) translateY(12px); }
  to   { opacity: 1; transform: scale(1) translateY(0); }
}
</style>