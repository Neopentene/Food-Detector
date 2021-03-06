<template>
  <div class="center-content">
    <button
      v-if="srcAdded"
      @click="setData(imgFile, bearer)"
      class="theme-btn rounded"
      style="padding: 0.4rem 0.5rem"
    >
      Detect Food Item
    </button>
    <div v-if="modalView" class="popUp-wrapper center-content">
      <div id="modal-wrapper" class="center-content column rounded">
        <div id="modal-header" class="center-content">
          <p>Image Results</p>
        </div>
        <div id="modal-body" class="center-content column">
          <div
            v-if="requestStatus && !isListEmpty"
            type="1"
            class="center-content column"
            style="padding: 0.5rem; position: absolute; top: 0; right: 0; left: 0"
          >
            <img
              :src="imgSrc"
              alt="image"
              style="max-width: 90%; max-height=18rem; padding: 1rem"
            />
            The image corresponds to the following dishes:
            <hr />
            <ul type="1" style="width: 90%">
              <li
                v-for="names in nameOfDishes"
                :key="names.name"
                style="word-break: break-word"
              >
                {{ names.name }}
              </li>
            </ul>
          </div>
          <div
            v-if="requestStatus && isListEmpty"
            class="center-content"
            style="text-align: center; padding: 0.5rem"
          >
            <p>{{ error }}</p>
          </div>

          <div v-if="loading" id="loading"></div>
        </div>
        <div id="modal-footer">
          <button @click="closeModal">Close</button>
        </div>
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
    bearer: String,
  },
})
export default class AnalyseImage extends Vue {
  private modalView: Boolean = false;
  private loading: Boolean = false;

  public Bearer: String = "";

  private requestStatus: Boolean = false;
  private isListEmpty: Boolean = true;

  private nameOfDishes: Array<any> = [];

  private error: String = "";

  private imageFile!: File;

  async analyseImageFromLogmeal(
    imageFile: File,
    Bearer: String
  ): Promise<{ results: Array<any>; object: any }> {
    return new Promise<{ results: Array<any>; object: any }>(function(
      resolve,
      reject
    ) {
      let xhr = new XMLHttpRequest(),
        formData = new FormData();
      formData.append("image", imageFile);
      xhr.onload = function() {
        resolve({
          results: <Array<any>>(
            JSON.parse(xhr.responseText!).recognition_results
          ),
          object: JSON.parse(xhr.responseText!),
        });
      };

      xhr.onerror = function() {
        reject({ results: [], object: JSON.parse(xhr.responseText!) });
      };

      xhr.onabort = function() {
        reject({ results: [], object: JSON.parse(xhr.responseText!) });
      };
      xhr.open("POST", "https://api.logmeal.es/v2/recognition/dish");
      xhr.setRequestHeader("Authorization", Bearer.toString());
      xhr.send(formData);
    });
  }

  async analyseImage() {
    this.loading = true;
    this.requestStatus = false;

    const result = await this.analyseImageFromLogmeal(
      this.imageFile,
      this.Bearer
    ).then(
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
    this.nameOfDishes = result.results;
    this.requestStatus = true;

    if (result.results == undefined) {
      if (result.object.message != undefined) {
        this.error = result.object.message;
      } else {
        this.error = "Some error occurred";
      }
    }

    this.loading = false;
  }

  setData(imageFile: File, Bearer: String) {
    this.imageFile = imageFile;
    this.Bearer = Bearer;
    this.showModal();
  }

  showModal() {
    this.modalView = true;
    this.analyseImage();
  }

  getBearer() {
    return this.Bearer;
  }

  closeModal() {
    this.modalView = false;
  }
}
</script>

<style scoped>
.popUp-wrapper {
  background: hsla(223, 43%, 8%, 0.9);
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
}

#modal-wrapper {
  width: 90%;
  max-width: 45rem;
  background-color: hsl(202, 12%, 73%);
  animation-name: fade-in;
  animation-fill-mode: forwards;
  animation-timing-function: ease-in;
  animation-duration: 350ms;
  opacity: 0;
}

#modal-wrapper * {
  color: hsl(223, 43%, 8%);
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
  position: relative;
  overflow: auto;
}

#modal-footer {
  padding: 0.5rem;
  width: 100%;
}

#modal-footer > button {
  padding: 0.35rem 0.5rem;
  border: none;
  margin: 0.5rem 1.5rem 0.5rem 0.5rem;
  float: right;
  border-radius: 0.35rem;
  background-color: hsl(208, 31%, 35%);
  color: hsl(86, 8%, 82%);
}

#modal-footer > button:hover {
  background-color: hsl(211, 34%, 27%);
}

#modal-footer > button:active {
  background-color: hsl(223, 43%, 8%);
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
    transform: translate(0%, -110%);
  }
  to {
    opacity: 1;
    transform: translate(0%, -10%);
  }
}
</style>
