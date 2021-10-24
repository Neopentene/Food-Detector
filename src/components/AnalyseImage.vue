<template>
  <div class="center-content">
    <button
      v-if="srcAdded"
      @click="setData(imgFile)"
      class="theme-btn rounded"
      style="padding: 0.4rem 0.5rem"
    >
      Detect Food Item
    </button>
    <div
      v-if="modalView"
      id="modal-wrapper"
      class="center-content column rounded"
    >
      <div id="modal-header" class="center-content">
        <p>Image Results</p>
      </div>
      <div id="modal-body" class="center-content column">
        <img v-if="srcAdded" :src="imgSrc" alt="Image" />
        <p v-else>No image uploaded</p>

        <div
          v-if="requestStatus && !isListEmpty"
          class="center-content column"
          style="text-align: center; padding: 0.5rem"
        >
          <h5>The image corresponds to the following dishes</h5>
          <ul id="itemList">
            <li v-for="names in nameOfDishes" :key="names.name">
              {{ names.name }}
            </li>
          </ul>
        </div>
        <div
          v-if="requestStatus && isListEmpty"
          class="center-content"
          style="text-align: center; padding: 0.5rem"
        >
          <p>The image uploaded is not a food item</p>
        </div>

        <div v-if="loading" id="loading"></div>
      </div>
      <div id="modal-footer">
        <button @click="closeModal">Close</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";

@Options({
  props: {
    imgSrc: String,
    imgFile: File,
    srcAdded: Boolean,
  },
})
export default class AnalyseImage extends Vue {
  modalView: Boolean = false;
  loading: Boolean = false;
  requestStatus: Boolean = false;
  isListEmpty: Boolean = true;
  nameOfDishes: Array<any> = [];
  private imageFile!: File;

  async analyseImageFromLogmeal(imageFile: File) {
    console.log(imageFile);
    return new Promise<{ results: Array<any> }>(function(resolve, reject) {
      let xhr = new XMLHttpRequest(),
        formData = new FormData();
      formData.append("image", imageFile);
      xhr.onload = function() {
        resolve({
          results: <Array<any>>(
            JSON.parse(xhr.responseText!).recognition_results
          ),
        });
      };

      xhr.onerror = function() {
        reject({ results: [] });
      };

      xhr.onabort = function() {
        reject({ results: [] });
      };
      xhr.open("POST", "https://api.logmeal.es/v2/recognition/dish");
      xhr.setRequestHeader(
        "Authorization",
        "Bearer c596d7293255a043c3bfbea974432951f5b4462a"
      );
      xhr.send(formData);
    });
  }

  async analyseImage() {
    this.loading = true;
    this.requestStatus = false;
    const result = await this.analyseImageFromLogmeal(this.imageFile).then(
      function(result) {
        return result;
      },
      function(result) {
        return result;
      }
    );

    if (result.results != undefined && result.results.length != 0) {
      this.isListEmpty = false;
    } else {
      this.isListEmpty = true;
    }
    this.nameOfDishes = result;
    this.requestStatus = true;
    this.loading = false;
    console.log(result, this.isListEmpty, this.requestStatus);
  }

  setData(imageFile: File) {
    this.imageFile = imageFile;
    this.showModal();
  }

  showModal() {
    this.modalView = true;
    this.analyseImage();
    //console.log(this.imageSrc);
  }
  closeModal() {
    this.modalView = false;
  }
}
</script>

<style scoped>
#modal-wrapper {
  width: 90%;
  max-width: 45rem;
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -100%);
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
  overflow-y: scroll;
  overflow-x: hidden;
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

#loading {
  display: inline-block;
  width: 50px;
  height: 50px;
  border: 3px solid hsl(47, 44%, 80%);
  border-radius: 50%;
  border-top-color: hsl(208, 31%, 35%);
  animation: spin 1s ease-in-out infinite;
  -webkit-animation: spin 1s ease-in-out infinite;
}

#itemList {
  list-style: none;
  text-align: center;
}

@keyframes spin {
  to {
    -webkit-transform: rotate(360deg);
  }
}
@-webkit-keyframes spin {
  to {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes fade-in {
  from {
    opacity: 0;
    transform: translate(-50%, -110%);
  }
  to {
    opacity: 1;
    transform: translate(-50%, -50%);
  }
}
</style>
