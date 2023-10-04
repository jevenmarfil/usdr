<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Blank</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <div>
        <p>
          Track function:
          <select v-model="selected">
            <option
              v-for="option in options"
              :key="option.text"
              :value="option"
            >
              {{ option.text }}
            </option>
          </select>
        </p>

        <qrcode-stream :track="selected.data" @error="logErrors" />
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from "vue";
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
} from "@ionic/vue";
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from "vue-qrcode-reader";

//state
const options: any = ref();
const selected = ref();
const logErrors = console.error;

//methods
const paintOutline = (detectedCodes: any, ctx: any) => {
  for (const detectedCode of detectedCodes) {
    const [firstPoint, ...otherPoints] = detectedCode.cornerPoints;

    ctx.strokeStyle = "red";

    ctx.beginPath();
    ctx.moveTo(firstPoint.x, firstPoint.y);
    for (const { x, y } of otherPoints) {
      ctx.lineTo(x, y);
    }
    ctx.lineTo(firstPoint.x, firstPoint.y);
    ctx.closePath();
    ctx.stroke();
  }
};

const paintBoundingBox = (detectedCodes: any, ctx: any) => {
  for (const detectedCode of detectedCodes) {
    const {
      boundingBox: { x, y, width, height },
    } = detectedCode;

    ctx.lineWidth = 2;
    ctx.strokeStyle = "#007bff";
    ctx.strokeRect(x, y, width, height);
  }
};

const paintCenterText = (detectedCodes: any, ctx: any) => {
  for (const detectedCode of detectedCodes) {
    const { boundingBox, rawValue } = detectedCode;

    const centerX = boundingBox.x + boundingBox.width / 2;
    const centerY = boundingBox.y + boundingBox.height / 2;

    const fontSize = Math.max(12, (50 * boundingBox.width) / ctx.canvas.width);
    console.log(boundingBox.width, ctx.canvas.width);

    ctx.font = `bold ${fontSize}px sans-serif`;
    ctx.textAlign = "center";

    ctx.lineWidth = 3;
    ctx.strokeStyle = "#35495e";
    ctx.strokeText(detectedCode.rawValue, centerX, centerY);

    ctx.fillStyle = "#5cb984";
    ctx.fillText(rawValue, centerX, centerY);
  }
};

options.value = [
  { text: "nothing (default)", data: undefined },
  { text: "outline", data: paintOutline },
  { text: "centered text", data: paintCenterText },
  { text: "bounding box", data: paintBoundingBox },
];

selected.value = options.value[1]

</script>

<style scoped></style>
