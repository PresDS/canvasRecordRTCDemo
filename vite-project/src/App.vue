<template>
  <h1>Record RTC - Canvas Recording Test</h1>
  <h2>Test 1 - Recording a canvas element</h2>
  <test-canvas @childRef="handleChildRef"></test-canvas>
  <div>
        <button @click="startRecording(passedCanvasRef)" :disabled="recording">Start Recording</button>
        <button @click="stopRecording" :disabled="!recording">Stop Recording</button>
  </div>

  <hr>

  <h2>Test 2 - Converting HTML to Canvas then recording</h2>

  <div class="example-2">
    <input v-model="userInput" @input="updateCanvas" ref="inputRef"/>
    <canvas ref="htmlToCanvasRef" class="converted-canvas"></canvas>
    <div>
        <button @click="startRecording(htmlToCanvasRef)" :disabled="recording">Start Recording</button>
        <button @click="stopRecording" :disabled="!recording">Stop Recording</button>
  </div>
  </div>



</template>

<script>
import RecordRTC from 'recordrtc';
import html2canvas from 'html2canvas';
import TestCanvas from './components/TestCanvas.vue';

export default {
  components: {
    TestCanvas,
  },
  data() {
    return {
      recording: false,
      recordRTC: null,
      canvasStream: null,
      passedCanvasRef: null,

      userInput: '',
      dynamicCanvas: null,

      htmlToCanvasRef: null,

    };
  },
  mounted() {
    this.htmlToCanvasRef = this.$refs.htmlToCanvasRef;
  },
  beforeDestroy() {
    clearInterval(this.intervalId);
  },
  methods: {
    handleChildRef(childRef) {
      this.passedCanvasRef = childRef
    },

    startRecording(canvasRef) {

      if(!canvasRef) {
        console.log('something went wrong with recording - canvasRef ', canvasRef);
        return;
      }

      const canvas = canvasRef;
      const stream = canvas.captureStream();

      // Create RecordRTC instance with the canvas stream
      this.recordRTC = RecordRTC(stream, {
        type: 'video',
      });

      // Start recording
      this.recordRTC.startRecording();
      this.recording = true;
    },
    stopRecording() {
      if (this.recordRTC) {
        // Stop recording
        this.recordRTC.stopRecording(() => {
          // Save the recorded video to a Blob
          const blob = this.recordRTC.getBlob();

          // Create a download link
          const url = URL.createObjectURL(blob);
          const a = document.createElement('a');
          document.body.appendChild(a);
          a.style = 'display: none';
          a.href = url;
          a.download = 'recorded-video.webm';

          // Trigger a click on the link to start the download
          a.click();

          // Clean up
          window.URL.revokeObjectURL(url);
          document.body.removeChild(a);

          this.recording = false;
        });
      }
    },

    updateCanvas() {
      let canvasRef = this.$refs.htmlToCanvasRef;

      // Use html2canvas to capture the input element and update the canvas
      html2canvas(this.$refs.inputRef).then((capturedCanvas) => {
        console.log('capturedCanvas', capturedCanvas)

        const dataURL = capturedCanvas.toDataURL();
        
        canvasRef.getContext('2d').drawImage(capturedCanvas, 0, 0);

        // const testAreaElement = document.querySelector('.example-2')

        // canvas.classList.add('converted-canvas')
        // // Clear existing canvas content
        // const context = canvas.getContext('2d');

        // let prevCanvas = document.querySelector('.converted-canvas')
        // // console.log('prevCanvas', prevCanvas)
        
        // if(prevCanvas) {
        //   testAreaElement.removeChild(prevCanvas);
        // }


        // context.clearRect(0, 0, canvas.width, canvas.height);

        // Draw the captured content onto the canvas
        // testAreaElement.appendChild(canvas);

        // canvasRef = canvas;

      });
    },

    
  },


}

</script>

<style>
body {
  background: rgb(49, 49, 51);
  color: rgba(1, 190, 48, 0.696);
  font-family: Arial, Helvetica, sans-serif;
}
</style>
