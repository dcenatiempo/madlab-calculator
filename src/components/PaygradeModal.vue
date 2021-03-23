<template>
<transition name="fade">
  <div v-if="isVisible" class="paygradModal overlay">
    <div class="card">
      <div class="header">
        <button @click="$emit('close')" class="close">X</button>
      </div>
      <img class="paygrade-image" src="../assets/madlab_paygrade.png" alt="MadLab Paygrade" />
    </div>
  </div>
</transition>
</template>

<script>

export default {
  name: 'PaygradeModal',
  props: {
    isVisible: {
      type: Boolean,
      default: false,
    },
    name: {
      type: String,
    }
  },
}
</script>

<style scoped lang="scss">
.paygradModal {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 909999;

  &.overlay {
    background: rgba(50, 50, 50, 0.5);
  }

  .card {
    background: white;
    border-radius: 10px;
    width: 95vw;
    max-width: 1200px;
    max-height: 95vh;

    .header {
      width: 100%;
      display: flex;
      justify-content: flex-end;
      .close {
        background: transparent;
        border: none;
        cursor: pointer;
        padding: 16px;
      }
    }
  }

  .paygrade-image {
    object-fit: contain;
    width: 100%;
  }
}
</style>
