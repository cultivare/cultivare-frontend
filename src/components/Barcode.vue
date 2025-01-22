<template>
    <v-container>
  
      <v-row>
        <v-col cols="12">
          <v-card class="d-flex align-center pa-3" color="grey-lighten-1">
            <canvas ref="barcodeCanvas" />
            <div style="margin-left: 20px;">
              <h2>{{ labelText }}</h2>
              <h4>{{ dateText }} </h4>
              <h4>{{ noteText }}</h4>
              <h6 v-if="showRestrictiveLabel">[ for microscopy use only ]</h6>
            </div>
          </v-card>
        </v-col>
      </v-row>
  
      <v-row>
        <v-col cols="8">
          <v-text-field v-model="noteText" label="Add Note"></v-text-field>
        </v-col>
        <v-col cols="4">
          <v-checkbox v-model="showRestrictiveLabel" label="Restrictive Label" />
        </v-col>
      </v-row>
      <v-row>
        <v-col> 
          <v-btn color="primary" prepend-icon="mdi-printer" @click="printToAPI">Print</v-btn>
        </v-col>
      </v-row>
    </v-container>
  </template>
  
  <script>
  import bwipjs from 'bwip-js';
  import api from "@/services/api";

  export default {
    props: {
      barcodeData: {
        type: Object,
        required: true,
      },
    },
    data() {
      return {
        barcodeType: this.barcodeData.barcodeType,
        barcodeTypes: ['qrcode', 'datamatrix', 'azteccode'],
        barcodeText: this.barcodeData.barcodeText,
        labelText: this.barcodeData.labelText,
        dateText: this.barcodeData.dateText,
        noteText: null,
        showRestrictiveLabel: true,
      };
    },
    mounted() {
      this.generateBarcode();
    },
    watch: {
      barcodeType() {
        this.generateBarcode();
      },
    },
    methods: {
      generateBarcode() {
        const canvas = this.$refs.barcodeCanvas;
        bwipjs.toCanvas(canvas, {
          bcid: this.barcodeType,
          text: this.barcodeText,
          scale: 2,
          monochrome: true
        });
      },
      printLabel() {
        window.print();
      },
      async printToAPI() {
        try {
          const response = await api.post('/labelprint/', {
            barcodeText: this.barcodeText,
            labelText: this.labelText,
            dateText: this.dateText,
            noteText: this.noteText,
            RestrictiveLabel: this.showRestrictiveLabel,
          });
          // Handle successful response here (e.g., show a success message)
          console.log('Print API response:', response);
        } catch (error) {
          // Handle error here (e.g., show an error message)
          console.error('Error printing to API:', error);
        }
      },
    },
  };
  </script>