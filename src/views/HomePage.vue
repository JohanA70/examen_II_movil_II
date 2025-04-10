<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Escáner de QR</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content class="ion-padding scanner-content">
      <ion-button @click="startScan" expand="block">
        Escanear Código QR
      </ion-button>

      <div v-if="isScanning" class="scanner-preview">
        <p class="scanner-instructions">Alinea el QR dentro de este marco</p>
      </div>

      <ion-list>
        <ion-list-header>Historial de Escaneos</ion-list-header>
        <ion-item v-for="(scan, index) in scanHistory" :key="index">
          <ion-label>{{ scan }}</ion-label>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonButton,
  IonList,
  IonListHeader,
  IonItem,
  IonLabel
} from '@ionic/vue';
import { ref } from 'vue';
import { CapacitorBarcodeScanner } from '@capacitor/barcode-scanner';

const scanHistory = ref<string[]>([]);
const isScanning = ref(false);

const startScan = async () => {
  try {
    // Iniciar escaneo
    const result = await CapacitorBarcodeScanner.scanBarcode({ hint: 17 });
    console.log('Permiso de cámara:', result);
    console.log('Resultado del escaneo:', result.ScanResult);

    if (result.ScanResult) {
      const scanContent = result.ScanResult;

      try {
        const parsed = JSON.parse(scanContent);
        console.log('Contenido JSON escaneado:', parsed);

        scanHistory.value.push(JSON.stringify(parsed));

        alert(`Datos recibidos: ${JSON.stringify(parsed, null, 2)}`);
      } catch (e) {

        if (scanContent.startsWith('http')) {
          window.open(scanContent, '_blank');
        } else if (scanContent.includes('@')) {
          window.location.href = `mailto:${scanContent}`;
        } else {
          alert(`Contenido escaneado: ${scanContent}`);
        }

        scanHistory.value.push(scanContent);
      }
    }
  } catch (error) {
    console.error('Error al escanear el código QR', error);
    alert('Error al escanear. Intente nuevamente.');
  }
};

</script>

<style scoped>
.scanner-content {
  --background: transparent;
  position: relative;
}

.scanner-preview {
  position: absolute;
  top: 120px;
  left: 50%;
  transform: translateX(-50%);
  width: 300px;
  height: 300px;
  border: 4px solid #00aaff;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  overflow: hidden;
  z-index: 10;
}

.scanner-instructions {
  position: absolute;
  bottom: 0;
  width: 100%;
  margin: 0;
  padding: 5px 0;
  text-align: center;
  background: rgba(0, 0, 0, 0.5);
  color: #fff;
  font-size: 14px;
}
</style>
