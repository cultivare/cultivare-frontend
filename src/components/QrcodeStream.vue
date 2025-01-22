<template>
    <v-card class="pa-6">
        <v-alert v-if="errorMessage" type="error" :text="errorMessage" />

        <v-card-title>
            Scanner
            <v-spacer />
        </v-card-title>
        <v-card-text>
            <qrcode-stream :track="track" :constraints="{ facingMode: selectedCamera }" @detect="onDetect"
                @init="onInit" :paused="isPaused" ref="qrcodeStream" autoplay muted>
            </qrcode-stream>
            <v-card-actions>
                    <v-btn prepend-icon="mdi-mushroom" v-if="showButton" color="success" @click="goToRoute"
                        size="x-large" elevation="10" variant="tonal">
                        {{ buttonName }}
                    </v-btn>
                    <v-spacer />
                    <v-btn prepend-icon="mdi-restart" v-if="isPaused" color="warning" @click="resetScan">
                        Scan more
                    </v-btn>
            </v-card-actions>

        </v-card-text>
    </v-card>
    <v-fab icon="mdi-camera-flip" class="ms-4 mb-4" elevation="10" @click="toggleCamera" color="red" location="right top start"
        size="64" absolute app appear></v-fab>
</template>


<script>
import { QrcodeStream } from 'vue-qrcode-reader';
import api from '@/services/api';

export default {
    components: {
        QrcodeStream,
    },
    data() {
        return {
            selectedCamera: 'environment',
            errorMessage: '',
            showButton: false,
            buttonName: '',
            itemId: null,
            isPaused: false, // Flag to control pausing the video stream
        };
    },
    methods: {
        toggleCamera() {
            this.selectedCamera =
                this.selectedCamera === 'environment' ? 'user' : 'environment';
        },
        async onDetect(result) {
            console.log('DETECTED', result);
            if (result.length > 0) {
                const decodedString = result[0].rawValue;
                this.isPaused = true; // Pause the video stream
                this.errorMessage = '';

                try {
                    const response = await api.get(`/cultures/${decodedString}`);
                    console.log("response", response)
                    this.itemId = response.data.id;
                    this.buttonName = response.data.name;
                    this.showButton = true;
                } catch (error) {
                    if (error.response) {
                        this.errorMessage = 'Culture ' + decodedString + ' not found';
                    } else if (error.request) {
                        this.errorMessage = 'No response from server';
                    } else {
                        this.errorMessage = error.message;
                    }
                }
            }
        },
        async onInit(promise) {
            try {
                await navigator.mediaDevices.getUserMedia({
                    video: { facingMode: this.selectedCamera },
                });

                await promise;
                this.errorMessage = '';
            } catch (error) {
                if (error.name === 'NotAllowedError') {
                    this.errorMessage = 'Hey! I need access to your camera';
                } else if (error.name === 'NotFoundError') {
                    this.errorMessage = 'Hmm, I couldn\'t find any camera';
                } else if (error.name === 'NotSupportedError') {
                    this.errorMessage =
                        'Seems like this page is served in non-secure context (HTTPS, localhost)';
                } else if (error.name === 'NotReadableError') {
                    this.errorMessage =
                        'Hey, is your camera being used by another app?';
                } else if (error.name === 'OverconstrainedError') {
                    this.errorMessage =
                        "Constraints don't match any installed camera. Did you requested the front camera although there is none?";
                } else if (error.name === 'StreamApiNotSupportedError') {
                    this.errorMessage = 'Stream API is not supported in this browser';
                } else if (error.name === 'InsecureContextError') {
                    this.errorMessage =
                        'Camera access is only permitted in secure context. Use HTTPS or localhost rather than HTTP.';
                } else {
                    this.errorMessage = 'Unknown error: ' + error.message;
                }
            }
        },
        goToRoute() {
            this.$router.push(`/cultures/${this.itemId}`);
        },
        track(detectedCodes, ctx) {
            if (detectedCodes.length > 0) {
                const detectedCode = detectedCodes[0];
                console.log(detectedCode);
                const [firstPoint, ...otherPoints] = detectedCode.cornerPoints;

                ctx.strokeStyle = 'red';
                ctx.beginPath();
                ctx.moveTo(firstPoint.x, firstPoint.y);
                for (const { x, y } of otherPoints) {
                    ctx.lineTo(x, y);
                }
                ctx.lineTo(firstPoint.x, firstPoint.y);
                ctx.closePath();
                ctx.stroke();

                const { boundingBox, rawValue } = detectedCode;
                const centerX = boundingBox.x + boundingBox.width / 2;
                const centerY = boundingBox.y + boundingBox.height / 2;
                const fontSize = Math.max(
                    12,
                    (50 * boundingBox.width) / ctx.canvas.width
                );

                ctx.font = `bold ${fontSize}px sans-serif`;
                ctx.textAlign = 'center';
                ctx.lineWidth = 3;
                ctx.strokeStyle = '#35495e';
                ctx.strokeText(rawValue, centerX, centerY);
                ctx.fillStyle = '#5cb984';
                ctx.fillText(rawValue, centerX, centerY);
            }
        },
        resetScan() {
            this.isPaused = false;
            this.errorMessage = '';
            this.showButton = false;
            this.itemId = null;
        },
    },
};
</script>
