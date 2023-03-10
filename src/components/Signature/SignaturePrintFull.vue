<template>
  <div v-show="!prints.Signature">
    <p class="text-center"><i>Kindly create your signature(s)</i></p>
    <button type="button" @click="createSignatureModal = true" class="btn btn-sm btn-primary d-block ms-auto mt-2">
      <span>Create</span>
    </button>
  </div>

  <div v-show="prints.Signature">
    <p class="text-center fw-normal">Pick a signature to append</p>
    <div class="grid qwerty">
      <label v-for="(print, index) in prints.Signature" :key="index" class="form-check-label border" :for="print.id">
        <div @click="
          getImgUrl({ category: print.category, type: print.type, print_id: print.id })
        ">
          <template v-if="print.user_id">
            <input type="radio" :name="print.id" v-model="selected" class="form-check-input tool_name" :id="print.id"
              :value="print.id" />
            <img :src="print.file" class="img-fluid" width="200" :alt="print.id" height="30" />
          </template>
        </div>
      </label>
    </div>

    <button type="button" class="btn btn-sm btn-primary d-block ms-auto mt-2" :disabled="!isDisabled"
      @click="uploadSignature">
      <span v-show="loading" class="spinner-border spinner-border-sm"></span>
      <span>Append</span>
    </button>
  </div>

  <ModalComp :show="createSignatureModal" :footer="false" @close="createSignatureModal = false">
    <template #header>
      <h4 class="modal-title">Create your signature</h4>
    </template>

    <template #body>
      <LeftTabWrapper>
        <LeftTabList title="Draw">
          <SignaturePad @close="createSignatureModal = false" />
        </LeftTabList>

        <LeftTabList title="Select">
          <SignatureSelectFull @close="createSignatureModal = false" />
        </LeftTabList>

        <LeftTabList title="Upload">
          <!-- <div class="row"> -->
          <div class="position-relative">
            <SignatureUpload @close="createSignatureModal = false" />
          </div>
          <!-- </div> -->
        </LeftTabList>
      </LeftTabWrapper>
      <div class="row">
        <div class="col-md-10 ms-auto mt-2">
          <p>
            By clicking create, I agree that all signatures, marks or initials created
            here are as valid as my hand written signature to the extent allowed by law.
          </p>
        </div>
      </div>
    </template>
  </ModalComp>
</template>

<script setup>
import ModalComp from "@/components/ModalComp.vue";
import LeftTabWrapper from "@/components/Tab/TabLeftNav/LeftTabWrapper.vue";
import LeftTabList from "@/components/Tab/TabLeftNav/LeftTabList.vue";
import SignaturePad from "@/components/Signature/SignaturePad.vue";
import SignatureSelectFull from "@/components/Signature/SignatureSelectFull.vue";
import SignatureUpload from "@/components/Signature/SignatureUpload.vue";

import { ref, defineEmits, watch } from "vue";
import { createNamespacedHelpers } from "vuex-composition-helpers/dist";
const { useGetters } = createNamespacedHelpers("print");

const { prints } = useGetters(["prints"]);

const createSignatureModal = ref(false);
const loading = ref(false);
const isDisabled = ref(false);
const selected = ref("");
const printId = ref("");

watch(
  () => prints.value,
  (newValue) => {
    prints.value = newValue;
  }
);

const toolClass = ref("");
const category = ref("");
const type = ref("");
const getImgUrl = (params) => {
  category.value = params.category;
  type.value = params.type;
  printId.value = params.print_id;
  toolClass.value = "tool-box main-element";
  isDisabled.value = true;
};

const emit = defineEmits(["selectedSignature"]);
const uploadSignature = () => {
  const uploadFile = {
    append_print_id: printId.value,
    tool_class: toolClass.value,
    category: category.value,
    type: type.value,
    tool_name: "Signature",
    tool_width: "95",
    tool_height: category.value == 'Type' ? '30' : "49",
  };

  emit("selectedSignature", uploadFile);
  emit("closeModal", true);

  loading.value = true;
  isDisabled.value = false;
  setTimeout(() => (loading.value = false), 500);
};
</script>

<style scoped>
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  place-items: center;
  gap: 20px;
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
  outline: 2px solid #003bb3;
}
</style>
