<template>
  <div class="w-100">
    <div data-aos="zoom-in">
      <div class="d-flex justify-content-between align-items-center text-start">
        <p class="small text-danger">
          Position yourself properly within the frame. <br />
          No glasses or face-cap. <br />
          Background must be plain.
        </p>
        <p data-aos="zoom-in" v-if="isTimer" class="timer">
          {{ timer }}
        </p>
      </div>

      <p v-if="isSelected == ''" class="text-center my-4"><i>Kindly select camera type to proceed</i></p>
      <template v-else>
        <template v-if="isLoading">
          <div class="text-center">
            <PreLoader class="my-4" />
          </div>
        </template>
      </template>

      <div v-show="isSelected != ''">
        <div v-show="!isLoading" class="video">
          <Camera :resolution="{ width: 160, height: 200 }" ref="camera" @loading="loading" @started="started"
            @camera-change="cameraChange" autoplay>
          </Camera>
        </div>
      </div>

      <div class="video-options my-1">
        <select class="form-select form-select-sm" v-model="isSelected" @change="changeCameraFace">
          <option value="">Select camera</option>
          <option v-for="(vDevice, index) in videoDevices" :value="vDevice.deviceId" :key="index">
            {{ vDevice.label }}
          </option>
        </select>
      </div>
    </div>

    <button v-if="isSelected != ''" class="btn btn-primary icon my-2" @click="snapshot" :disabled="spinner">
      <span class="iconify" data-icon="fluent:camera-20-filled">
        <span v-show="spinner" class="spinner-border spinner-border-sm text-white"></span>
        <Icon v-show="!spinner" icon="mdi-light:camera" />
      </span>
    </button>

    <template v-if="prints.Photograph">
      <div class="grid grid__3 mt-2">
        <label v-for="(photo, index) in prints.Photograph" :key="index" class="form-check-label border custom-width"
          :for="photo.created_at">
          <div @click="getPrintId({ category: 'Upload', print_id: photo.id })">
            <template v-if="photo.user_id">
              <div class="position-relative">
                <input type="radio" name="photo" v-model="selected" class="form-check-input tool_name pass"
                  :id="photo.created_at" :value="photo.id" />

                <img :src="photo.file" class="img-fluid" :alt="photo.id" />
                <a role="button" @click="deletePassport(photo.id)"
                  class="text-danger btn-close d-block text-end delete"></a>
              </div>
            </template>
          </div>
        </label>
      </div>

      <button type="button" class="btn btn-sm btn-primary d-block ms-auto mt-2" :class="{ disabled: !isDisabled }"
        @click="affixSnap">
        <span v-show="isLoading" class="spinner-border spinner-border-sm"></span>
        <span>Append</span>
      </button>
    </template>
  </div>

  <ModalComp :show="isDelete" :size="'modal-sm'" @close="isDelete = false">
    <template #header>
      <h4 class="modal-title text-danger">
        <Icon icon="eva:alert-triangle-outline" style="margin-bottom: 3px" />
        Alert
      </h4>
    </template>

    <template #body>
      <h5 class="text-center">Are you sure?</h5>
      <p class="fw-normal text-center"><i>You won't be able to revert this!</i></p>
    </template>

    <template #footer>
      <button type="button" class="btn btn-sm btn-primary d-block ms-auto" :class="{ disabled: spinner }"
        @click="proceedToDelete">
        <span v-show="spinner" class="spinner-border spinner-border-sm"></span>
        <span>Proceed</span>
      </button>
    </template>
  </ModalComp>
</template>

<script setup>
import Camera from "simple-vue-camera";
import PreLoader from "@/components/PreLoader.vue";
import ModalComp from "@/components/ModalComp.vue";
import { Icon } from "@iconify/vue";
import AOS from "aos";
import { ref, defineEmits, watch } from "vue";

import { useActions, useGetters } from "vuex-composition-helpers/dist";

const { prints, isSaved } = useGetters({
  prints: "print/prints",
  isSaved: "print/isSaved",
});

const { savePrint, removePrint } = useActions({
  savePrint: "print/savePrint",
  removePrint: "print/removePrint",
});

const spinner = ref(false);
const isLoading = ref(false);
const camera = ref("");
const timer = ref(null);
const isTimer = ref(false);
const isDisabled = ref(false);
const isDelete = ref(false);
const printId = ref("");
const isSelected = ref('')
const videoDevices = ref([])
const emit = defineEmits(["close"]);

watch(
  () => isSaved.value,
  (newSave) => {
    if (newSave) {
      isSaved.value = newSave;
    }
  }
);

const loading = () => (spinner.value = isLoading.value = true);

const started = () => {
  spinner.value = isLoading.value = false
  AOS.init({ duration: 500 })
};

const getPrintId = (params) => {
  isDisabled.value = true;
  printId.value = params.print_id;
};

const affixSnap = () => {
  const uploadPassport = { append_print_id: printId.value };

  emit("affix", uploadPassport);

  isLoading.value = true;
  isDisabled.value = false;
  setTimeout(() => { isLoading.value = false }, 1000);
};

const deletePassport = (params) => {
  isDelete.value = true;
  printId.value = params;
};

const proceedToDelete = () => {
  isDisabled.value = spinner.value = true;
  removePrint(printId.value);

  setTimeout(() => {
    isDisabled.value = spinner.value = isDelete.value = false;
  }, 1000);
};

setTimeout(async () => {
  const devices = await camera.value?.devices();
  videoDevices.value = devices.filter(device => device.kind === 'videoinput');
}, 3000);

const changeCameraFace = () => camera.value?.changeCamera(isSelected.value);

const snapshot = async () => {
  spinner.value = isTimer.value = true;
  let counter = 3;
  timer.value = 3

  await camera.value?.snapshot({ width: 160, height: 200 }, "image/png", 0.5);
  const dataUrl = camera.value?.canvas.toDataURL("image/png");
  const uploadPassport = {
    file: dataUrl,
    type: "Photograph",
    category: "Upload",
  };

  const interval = setInterval(function () {
    counter--;
    timer.value = counter
    if (counter < 0) {
      clearInterval(interval);
      spinner.value = isTimer.value = false
      savePrint(uploadPassport);
    }
  }, 1000);
};
</script>

<style scoped>
.grid {
  display: grid;
  place-items: center;
  gap: 15px;
  width: 50%;
  margin: 0 auto;
}

.grid__3 {
  grid-template-columns: repeat(3, 1fr);
}

.video-options {
  width: 160px;
  margin: 0 auto;
}

.custom-width {
  width: 100px
}

.icon {
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  padding: 20px;
  text-align: center;
  font-size: 1.5rem;
  margin: 0 auto;
  height: 35px;
  width: 35px;
}

.timer {
  border-radius: 50%;
  width: 80px;
  height: 80px;
  font-size: 2rem;
  background: var(--bs-purple);
  color: #fff;
  padding: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.video {
  width: 160px;
  height: 200px;
  margin: 0 auto;
  -webkit-transform: scaleX(-1);
  transform: scaleX(-1);
}

[type="radio"] {
  position: absolute;
  opacity: 0;
}

[type="radio"]+img {
  cursor: pointer;
  margin-right: 0.5rem;
}

[type="radio"]+img:hover {
  outline: 0.5px solid #003bb3;
}

[type="radio"]:checked+img {
  outline: 3px solid #003bb3;
}

.delete {
  position: absolute;
  top: -14px;
  right: -14px;
  border-radius: 50%;
  border: 1px solid red;
  padding: 8px;
}

@media screen and (max-width: 991.5px) {
  .timer {
    width: 30px;
    height: 30px;
    font-size: 1rem;
  }

  .grid {
    width: 100%;
  }

  .grid__3 {
    grid-template-columns: repeat(3, 1fr);
  }

  .custom-width {
    width: 80px
  }
}
</style>
