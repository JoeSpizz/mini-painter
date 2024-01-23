<template>
  <div class="image-upload-container">
    <input type="file" @change="onFileChange" />
    <canvas v-if="image" ref="canvas" @mousedown="startPainting" @mousemove="paint" @mouseup="stopPainting" @mouseleave="stopPainting"></canvas>
    <input type="color" v-model="brushColor" /> <!-- Color Picker -->
    <input type="range" v-model="brushSize" min="1" max="50" /> <!-- Brush Size Selector -->
  </div>
</template>
  
  <script>
  export default {
    data() {
      return {
        image: null,
        isPainting: false,
        brushColor: '#000000', // Default brush color, can be changed later
        brushSize: 5, // Default brush size
      };
    },
    methods: {
      onFileChange(e) {
        const file = e.target.files[0];
        if (!file) return;
  
        const reader = new FileReader();
        reader.onload = (e) => {
            this.image = e.target.result;
            this.$nextTick(() => {
            this.loadImageOntoCanvas();
            });
        };
        reader.readAsDataURL(file);
      },
      loadImageOntoCanvas() {
        const canvas = this.$refs.canvas;
        if (!canvas) return;
        const ctx = canvas.getContext('2d');
        const img = new Image();
        img.onload = () => {
            // Calculate the scaling factor to resize the image
            const screenFactor = 0.8; // Use 80% of the screen size
            const maxWidth = window.innerWidth * screenFactor;
            const maxHeight = window.innerHeight * screenFactor;
            const scale = Math.min(maxWidth / img.width, maxHeight / img.height);

            // Set canvas size based on the scaled image size
            canvas.width = img.width * scale;
            canvas.height = img.height * scale;

            // Draw the image on the canvas at the resized dimensions
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        };
        img.src = this.image;
        },
    startPainting(event) {
      this.isPainting = true;
      this.paint(event);
    },
    paint(event) {
  if (!this.isPainting) return;
  const ctx = this.$refs.canvas.getContext('2d');
  const rect = this.$refs.canvas.getBoundingClientRect();

  // Adjusted coordinates for brush stroke
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;

  ctx.strokeStyle = this.brushColor;
  ctx.lineWidth = this.brushSize;
  ctx.lineCap = 'round';

  ctx.lineTo(x, y);
  ctx.stroke();
  ctx.beginPath();
  ctx.moveTo(x, y);
},
    stopPainting() {
      this.isPainting = false;
      this.$refs.canvas.getContext('2d').beginPath();
    },
  }
};
</script>
  <style scoped>
  .image-upload-container {
    display: flex;
    flex-direction: column; /* Stack children vertically */
    align-items: center; /* Center children horizontally */
    gap: 10px; /* Add space between elements */
  }
  
  canvas {
    max-width: 100%; /* Ensure canvas doesn't overflow container */
  }
  </style>