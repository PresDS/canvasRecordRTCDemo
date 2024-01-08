<template>
    <canvas ref="canvas" width="200" height="200"></canvas>
</template>

<script>
export default {
  data() {
    return {
      colors: ["red", "blue"],
      currentColorIndex: 0,
      intervalId: null,
      canvasRef: null,
    };
  },
  mounted() {

    this.canvasRef = this.$refs.canvas
    this.passRefToParent()

    this.drawCircle();
    this.startColorChanging();

  },
  beforeDestroy() {
    clearInterval(this.intervalId);
  },
  methods: {
    passRefToParent() {
      this.$emit('childRef', this.canvasRef);
    },
    drawCircle() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      const centerX = canvas.width / 2;
      const centerY = canvas.height / 2;
      const radius = 50;
      
      ctx.strokeStyle = "black"; // Border color
      ctx.lineWidth = 2; // Border width

      ctx.beginPath();
      ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI);
      ctx.fillStyle = this.colors[this.currentColorIndex];
      ctx.fill();
    },
    startColorChanging() {
      this.intervalId = setInterval(() => {
        this.currentColorIndex = (this.currentColorIndex + 1) % this.colors.length;
        this.drawCircle();
      }, 1000);
      
    },
  },
};
</script>

<style scoped>
canvas {
  border: 2px black solid;
  background: rgb(195, 247, 178);
}
</style>
