<template>
  <transition name="fade">
    <div class="modal" v-if="show">
      <div class="modal__backdrop" @click="closeModal()" />

      <div class="modal__dialog">
        <div class="modal__header">
          <slot name="header" />
        </div>

        <div class="modal__body">
          <slot name="body" />
        </div>

         <div class="modal__header1">
          <slot name="header1" />
        </div>

        <div class="modal__body1">
          <slot name="body1" />
        </div>

        <div class="modal__footer">
          <slot name="footer" />
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  name: "Modal",
  data() {
    return {
      show: false,
    };
  },
  methods: {
    closeModal() {
      this.show = false;
      document.querySelector("main").classList.remove("overflow-hidden");
    },
    openModal() {
      this.show = true;
      document.querySelector("main").classList.add("overflow-hidden");
    },
  },
};
</script>


<style scoped>
.modal {
  position: fixed;
  top: 15%;
  right: 15%;
  bottom: 15%;
  left: 15%;
  z-index: 20000;
  overflow-x: hidden;
  overflow-y: auto;
  background-color:#1b1b19;
  border-radius: 6px;
  padding: auto;
  color: #898883;
  
}
.backdrop {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 1;
  border-radius: 10px;
  background-color: #1b1b19;
}
.modal__dialog {
  background-color:#1b1b19;
  border-radius: 5px;
  margin: 20px;
  display: flex;
  flex-direction: column;
  z-index: 2;
  padding: 20px;
  left: 20px;
  top: 10px;
    border-radius: 6px;
  color: #898883;
  @media screen and (max-width: 992px) {
    width: 70%;
  }

}
.close {
  width: 30px;
  height: 30px;
  background-color: black;
}
.header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding-left: 100px;
  font-family: "khula";
}
.body {
  overflow: auto;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  font-family: "Biryani";
}
.footer {
  font-family: "khula";
  background-color: black;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
