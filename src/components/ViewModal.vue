<template>
  <div id="modal-wrapper" class="center-content column rounded">
    <div id="modal-header" class="center-content">
      <p>Image Preview</p>
    </div>
    <div id="modal-body" class="center-content">
      <img v-if="srcAdded" :src="imgSrc" alt="Image" />
      <p v-else>No image was uploaded</p>
    </div>
    <div id="modal-footer">
      <button @click="closeModal">Close</button>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";

@Options({
  props: {
    imgSrc: String,
    srcAdded: Boolean,
  },
})
export default class ViewModal extends Vue {
  closeModal() {
    this.$emit("close");
  }
}
</script>

<style scoped>
#modal-wrapper {
  width: 90%;
  max-width: 45rem;
  position: absolute;
  top: 20%;
  left: 50%;
  transform: translate(-50%, -110%);
  background-color: hsl(47, 44%, 89%);
  animation-name: fade-in;
  animation-fill-mode: forwards;
  animation-timing-function: ease-in;
  animation-duration: 500ms;
  opacity: 0;
}

#modal-wrapper * {
  color: hsl(208, 31%, 35%);
}

#modal-header,
#modal-body {
  width: 100%;
  border-bottom: 1px solid hsl(223, 43%, 8%);
}

#modal-header > p {
  margin: 0;
  padding: 0.75rem;
  font-size: large;
}

#modal-body {
  height: 20rem;
  max-height: 20rem;
}

#modal-body > img {
  padding: 0.25rem;
  max-width: 100%;
  max-height: 18rem;
}

#modal-footer {
  padding: 0.5rem;
  width: 100%;
}

#modal-footer > button {
  padding: 0.25rem;
  margin: 0.5rem 1.5rem 0.5rem 0.5rem;
  float: right;
  border-radius: 0.35rem;
  background-color: hsl(47, 44%, 89%);
  border: 2px solid hsl(208, 31%, 35%);
}

#modal-footer > button:hover {
  border-color: hsl(223, 43%, 8%);
}

#modal-footer > button:active {
  background-color: hsl(47, 44%, 80%);
}

@keyframes fade-in {
  from {
    opacity: 0;
    transform: translate(-50%, -110%);
  }
  to {
    opacity: 1;
    transform: translate(-50%, 0%);
  }
}
</style>
