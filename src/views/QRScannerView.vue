<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Blank</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <div>
        <p class="decode-result">
          Last result: <b>{{ result }}</b>
        </p>

        <qrcode-stream
          :paused="paused"
          @detect="onDetect"
          @error="onError"
          @camera-on="resetValidationState"
        >
          <div v-if="validationSuccess" class="validation-success">
            {{ result }}
          </div>

          <div v-if="validationFailure" class="validation-failure">
            This is NOT a URL!
          </div>

          <div v-if="validationPending" class="validation-pending">
            Long validation in progress...
          </div>
        </qrcode-stream>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { computed, ref } from "vue";
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
} from "@ionic/vue";
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from "vue-qrcode-reader";

const isValid = ref(undefined);
const paused = ref(false);
const result:any = ref(null);

const validationPending = computed(
  () => isValid.value === undefined && paused.value
);
const validationSuccess = computed(() => isValid.value === true);
const validationFailure = computed(() => isValid.value === false);

const onError = console.error;

const resetValidationState = () => {
  isValid.value = undefined;
};

const onDetect = async ([firstDetectedCode]: any) => {
  result.value = firstDetectedCode.rawValue;
  paused.value = true;

  // pretend it's taking really long
  await timeout(3000);
  if (result.value) {
    isValid.value = result.value.startsWith("http");
  }

  // some more delay, so users have time to read the message
  await timeout(2000);
  paused.value = false;
};

const timeout = (ms: number) => {
  return new Promise((resolve) => {
    window.setTimeout(resolve, ms);
  });
};
</script>

<style scoped>
.validation-success,
.validation-failure,
.validation-pending {
  position: absolute;
  width: 100%;
  height: 100%;

  background-color: rgba(255, 255, 255, 0.8);
  padding: 10px;
  text-align: center;
  font-weight: bold;
  font-size: 1.4rem;
  color: black;

  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
}
.validation-success {
  color: green;
}
.validation-failure {
  color: red;
}
</style>
