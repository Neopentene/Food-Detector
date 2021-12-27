<template>
  <span id="title" class="center-content">
    <span> </span>
    <div class="center-text">
      <h1>{{ title[0].toUpperCase() }}</h1>
      <h3>{{ title.slice(1, title.length) }}</h3>
    </div>
    <span> </span>
  </span>

  <div class="center-content p-5P relative column" style="padding-bottom: 1rem">
    <div id="upload-wrapper" class="center-content column p-5P rounded">
      <h3 style="text-align: center; color: hsl(202, 12%, 73%)">
        Upload image of any food item
      </h3>
      <div id="custom-input">
        <input
          @change="uploadState()"
          type="file"
          name="food-img"
          id="food-img"
          hidden
          aria-hidden="true"
          :accept="fileExtensions"
          ref="foodImg"
        />
        <label for="food-img" id="food-img-upload" class="center-content">{{
          labelState
        }}</label>
        <p>{{ fileString }}</p>
      </div>
      <div
        @click="resetUpload"
        class="theme-btn bot-rounded center-content"
        style="padding: 0.4rem; font-size: 0.85rem;"
        :style="{ opacity: +srcAdded ? 1 : 0 }"
      >
        Clear Upload
      </div>
      <div class="p-5P">
        <button
          @click="showModal"
          class="theme-btn rounded"
          style="padding: 0.4rem 0.5rem"
        >
          View Image
        </button>
      </div>
    </div>
    <div v-if="error" class="center-content alert">
      <span>{{ errorText }}</span>
      <span class="closebtn rounded" @click="closeError">&times;</span>
    </div>
  </div>
  <view-modal
    @close="showModal"
    v-if="modalView"
    :imgSrc="imgSrc"
    :srcAdded="srcAdded"
  />
  <analyse-image
    v-if="srcAdded"
    :imgFile="imageFile"
    :imgSrc="imgSrc"
    :srcAdded="srcAdded"
    :bearer="Bearer"
  />
</template>

<script lang="ts">
import { Options, Vue } from "vue-class-component";
import ViewModal from "./components/ViewModal.vue";
import AnalyseImage from "./components/AnalyseImage.vue";

@Options({
  components: {
    ViewModal,
    AnalyseImage,
  },
})
export default class App extends Vue {
  title: String = "Food Detector";
  fileExtensions: String = "image/*";

  labelState: String = "Upload Image";
  fileString: String = "No file has been selected";

  imgSrc: String | ArrayBuffer = "";

  srcAdded: Boolean = false;
  modalView: Boolean = false;
  error: Boolean = false;

  errorText: String = "No Errors";

  Bearer: String = "Bearer d7a57b22d03f660fd972df488fc5f6a8287b8c93";

  private imageFile!: File;

  async getFileData(): Promise<{
    result: String | ArrayBuffer;
    name: String;
    type: String;
  }> {
    const file: File = (<HTMLInputElement>this.$refs.foodImg).files![0];
    return new Promise<{
      result: String | ArrayBuffer;
      name: String;
      type: String;
      file: File | null;
    }>(function(resolve, reject) {
      if (file) {
        const fileReader = new FileReader();

        fileReader.onload = function() {
          resolve({
            result: this.result!,
            name: file.name,
            type: file.type,
            file: file,
          });
        };

        fileReader.onerror = function() {
          reject({
            result: "",
            name: "No file has been selected",
            type: "",
            file: null,
          });
        };

        fileReader.onabort = function() {
          reject({
            result: "",
            name: "No file has been selected",
            type: "",
            file: null,
          });
        };

        fileReader.readAsDataURL(file);
      }
    });
  }

  async uploadState() {
    const result: {
      result: String | ArrayBuffer;
      name: String;
      type: String;
      file: File | null;
    } = await this.getFileData().then(
      function(imageSrc) {
        return imageSrc;
      },
      function(imageSrc) {
        return imageSrc;
      }
    );

    this.imgSrc = result.result;
    if (this.imgSrc != "" && /^image\//gm.test(result.type.toString())) {
      this.fileString = result.name;
      try {
        this.imageFile = await this.compressOrSetImageFile(result).then(
          function(file) {
            return file;
          },
          function() {
            throw "Some error occurred";
          }
        );
      } catch (error) {
        this.fileString = "No file has been selected";
        this.labelState = "Upload Image";
        this.srcAdded = false;
        this.setError("The Uploaded File seems to be corrupted or incorrect");
        throw error;
      }
      this.labelState = "Image Uploaded";
      this.srcAdded = true;
    } else {
      this.imgSrc = "";
      this.fileString = "No file has been selected";
      this.labelState = "Upload Image";
      this.srcAdded = false;
      this.setError(
        "Please select an image not (" + result.type.split("/")[1] + " file)"
      );
    }
  }

  async compressOrSetImageFile(result: {
    result: String | ArrayBuffer;
    name: String;
    type: String;
    file: File | null;
  }): Promise<File> {
    return new Promise<File>(function(resolve, reject) {
      const img = <HTMLImageElement>document.createElement("img");
      const imgURL = result.result.toString();

      img.onload = function() {
        const canvas = <HTMLCanvasElement>document.createElement("canvas");
        canvas.height = img.naturalHeight;
        canvas.width = img.naturalWidth;

        const ctx = <CanvasRenderingContext2D>canvas.getContext("2d");

        ctx.drawImage(img, 0, 0);

        const sizeRatio =
          result.file?.size != undefined && result.file.size > 1000000
            ? 1000000 / result.file.size
            : 0.7;

        /*
        // Code to download the image
        console.log(canvas.toDataURL("image/jpeg", sizeRatio));
        const imageDownload = <HTMLAnchorElement>document.createElement("a");
        imageDownload.href = canvas.toDataURL("image/jpeg", sizeRatio);
        imageDownload.target = "_blank";
        const nameArray = result.file
          ? result.file.name.split(".")
          : "image.jpg".split(".");
        let name = "";
        for (let pos = 0; pos < nameArray.length - 1; pos++) {
          name += nameArray[pos] + ".";
        }
        imageDownload.download = name + "jpg";
        imageDownload.click();
        */

        canvas.toBlob(
          function(blob) {
            const file = new File([blob ? blob : new Blob()], "image.jpeg", {
              type: "image/jpeg",
              lastModified: new Date().getDate(),
            });
            resolve(file);
          },
          "image/jpeg",
          sizeRatio
        );
      };

      img.onerror = function() {
        reject();
      };

      img.onabort = function() {
        reject();
      };

      img.src = imgURL;
    });
  }

  setError(msg: String) {
    this.errorText = msg;
    this.error = true;
  }

  resetUpload() {
    if (this.srcAdded == false) {
      this.setError("No image was uploaded");
      return;
    }
    this.imgSrc = "";
    this.fileString = "No file has been selected";
    this.labelState = "Upload Image";
    this.srcAdded = false;
    (<HTMLInputElement>this.$refs.foodImg).value = "";
  }

  closeError() {
    this.error = false;
  }

  showModal() {
    this.modalView = !this.modalView;
  }
}
</script>

<style>
/*
* 
* font-family: Avenir, Helvetica, Arial, sans-serif;
* ---- Color Palette ----
* Darker Rick Black: hsl(223, 43%, 8%);
* Rich Black: hsl(222, 43%, 9%);
* Prussian Blue: hsl(215, 40%, 19%);
* Indigo Dye: hsl(211, 34%, 27%);
* Black Coral: hsl(208, 31%, 35%);
* Shadow Blue: hsl(214, 25%, 56%);
* Eggshell: hsl(47, 44%, 89%);
* Silver Sand: hsl(202, 12%, 73%);
* Lavender Blush: hsl(333, 21%, 92%);
* Mountbatten Pink: hsl(307, 13%, 58%);
* Timberwolf: hsl(86, 8%, 82%);
* 
*/

:root {
  background-color: hsl(223, 43%, 8%);
  font-family: Avenir, Helvetica, Arial, sans-serif;
}

.flex-display {
  display: flex;
}

.center-content {
  display: flex;
  justify-content: center;
  align-items: center;
}

.center-text {
  text-align: center;
}

.mt-2P {
  margin-top: 2px;
}

.m-2 {
  margin: 2rem;
}

.m-2-x-b {
  margin: 2rem 2rem 0 2rem;
}

.p-2 {
  padding: 2rem;
}

.p-5P {
  padding: 5%;
}

.fit-view {
  height: 100%;
  width: 100%;
}

.row {
  flex-direction: row;
}

.column {
  flex-direction: column;
}

.relative {
  position: relative;
}

.bot-rounded {
  border-bottom-left-radius: 0.35rem;
  border-bottom-right-radius: 0.35rem;
}

.rounded {
  border-radius: 0.35rem;
}

#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: hsl(223, 43%, 8%);
  color: #2c3e50;
}

#title {
  border-bottom: 2px solid hsl(214, 25%, 56%);
}

#title > div {
  margin: 0 5% 0 5%;
}

#title > div * {
  display: inline;
  margin: 0;
  color: hsl(202, 12%, 73%);
  background-color: transparent;
}

#title > div > h1 {
  font-size: 3.5rem;
}

#title > div > h3 {
  font-family: Cambria, Cochin, Georgia, Times, "Times New Roman", serif;
  font-size: 2.25rem;
}

#title > span {
  height: 5%;
  width: 10%;
  border: 0.25rem solid hsl(214, 25%, 56%);
  background-color: hsl(214, 25%, 56%);
}

#upload-wrapper {
  width: 100%;
  max-width: 40rem;
  box-sizing: border-box;
  border: 2px solid hsl(208, 31%, 35%);
}

#custom-input {
  width: 100%;
  display: flex;
  flex-direction: row;
  border-radius: 0.25rem;
  overflow: hidden;
  padding: 0;
  box-shadow: 0 0 0 2px hsl(215, 40%, 19%);
}

#custom-input > p {
  margin: auto;
  padding: 0.25rem;
  min-width: 0;
  width: 70%;
  font-size: 1em;
  text-align: center;
  overflow-x: hidden;
  text-overflow: ellipsis;
  color: hsl(214, 25%, 56%);
}

#food-img-upload {
  width: 30%;
  min-width: min-content;
  margin: 0;
  padding: 0.25rem;
  text-align: center;
  font-size: 0.9em;
  border-top-left-radius: 0.25rem;
  border-bottom-left-radius: 0.25rem;
  background-color: hsl(215, 40%, 19%);
  color: hsl(333, 21%, 92%);
  user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}

#food-img-upload:hover {
  background-color: hsl(208, 31%, 35%);
}

#food-img-upload:active {
  background-color: hsl(211, 34%, 27%);
}

.alert {
  margin: 2%;
  padding: 0.75rem;
  position: absolute;
  top: 0;
  border-radius: 0.25rem;
  background-color: hsl(214, 25%, 56%);
  color: hsl(333, 21%, 92%);
}

.closebtn {
  margin-left: 1rem;
  padding: 0 0.6rem 0.3rem 0.5rem;
  color: hsl(202, 12%, 73%);
  font-weight: bold;
  float: right;
  font-size: 1.5rem;
  cursor: pointer;
  transition: 250ms;
}

.closebtn:hover {
  color: hsl(215, 40%, 19%);
  box-shadow: 0 0 0 2px hsl(215, 40%, 19%);
}

.theme-btn {
  background-color: hsl(222, 43%, 9%);
  border: 2px solid hsl(215, 40%, 19%);
  font-size: 1rem;
  text-align: center;
  color: hsl(202, 12%, 73%);
  user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
}

.theme-btn:hover {
  background-color: hsl(215, 40%, 19%);
}

.theme-btn:active {
  background-color: hsl(208, 31%, 35%);
  border-color: hsl(208, 31%, 35%);
}
</style>
