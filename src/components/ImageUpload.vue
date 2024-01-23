<template>
  <div class="image-upload-container">
    <input type="file" @change="onFileChange" />
    <canvas v-if="image" ref="canvas" @mousedown="startPainting" @mousemove="paint" @mouseup="stopPainting" @mouseleave="stopPainting"></canvas>
    <input type="color" v-model="brushColor" /> <!-- Color Picker -->
    <input type="range" v-model="brushSize" min="1" max="50" /> <!-- Brush Size Selector -->
    <button @click="saveAsPNG">Save as PNG</button>
    <div class="canvas-container" ref="canvasContainer">
      <canvas ref="baseCanvas" v-if="image"></canvas> <!-- Base Layer -->
      <canvas ref="paintCanvas" v-if="image" @mousedown="startPainting" @mousemove="paint" @mouseup="stopPainting" @mouseleave="stopPainting"></canvas> <!-- Paint Layer -->
    </div>
    <button @click="toggleEraser">Toggle Eraser</button>
    <button @click="undo">Undo</button>
    <button @click="redo">Redo</button>
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
        usingEraser: false,
        history: [],
        historyStep: -1,
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
        const baseCanvas = this.$refs.baseCanvas;
        const paintCanvas = this.$refs.paintCanvas;
        const ctx = baseCanvas.getContext('2d');
        const img = new Image();

        img.onload = () => {
            // Calculate the scaling factor to resize the image
            const screenFactor = 0.8; // Use 80% of the screen size
            const maxWidth = window.innerWidth * screenFactor;
            const maxHeight = window.innerHeight * screenFactor;
            const scale = Math.min(maxWidth / img.width, maxHeight / img.height);


            baseCanvas.width = paintCanvas.width = img.width * scale;
            baseCanvas.height = paintCanvas.height = img.height * scale;

            ctx.drawImage(img, 0, 0, baseCanvas.width, baseCanvas.height);
            this.pushToHistory(); // Push initial state after image load
      };

      img.src = this.image;
    },
    startPainting(event) {
      this.isPainting = true;
      this.paint(event);
    },
    saveAsPNG() {
      const canvas = this.$refs.canvas;
      const image = canvas.toDataURL("image/png").replace("image/png", "image/octet-stream");
      const link = document.createElement('a');
      link.download = 'mini-painter.png';
      link.href = image;
      link.click();
    },
    toggleEraser() {
      this.usingEraser = !this.usingEraser;
    },
    paint(event) {
        if (!this.isPainting) return;
        const ctx = this.$refs.canvas.getContext('2d');
        const rect = this.$refs.canvas.getBoundingClientRect();

        // Adjusted coordinates for brush stroke
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;

        // Set the eraser or brush properties
        if (this.usingEraser) {
            ctx.globalCompositeOperation = 'destination-out'; // Eraser mode
            ctx.lineWidth = this.brushSize * 2; // Eraser size could be different
        } else {
            ctx.globalCompositeOperation = 'source-over'; // Normal drawing mode
            ctx.strokeStyle = this.brushColor;
            ctx.lineWidth = this.brushSize;
        }
        ctx.lineCap = 'round';

        ctx.lineTo(x, y);
        ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(x, y);
        },
    stopPainting() {
        this.isPainting = false;
        const ctx = this.$refs.canvas.getContext('2d');
        ctx.beginPath();
        this.pushToHistory(); // Save the state after finishing the painting stroke
        },
    pushToHistory() {
        if (this.historyStep < this.history.length - 1) {
            this.history = this.history.slice(0, this.historyStep + 1);
        }
        const canvas = this.$refs.canvas;
        console.log("Saving history state", this.historyStep, canvas.toDataURL().substring(0, 30));
        this.history.push(canvas.toDataURL());
        this.historyStep++;
        },
    undo() {
        if (this.historyStep > 0) {
            this.historyStep--;
            this.loadFromHistory();
        }
        },
    redo() {
        if (this.historyStep < this.history.length - 1) {
            this.historyStep++;
            this.loadFromHistory();
        }
        },
    loadFromHistory() {
        const img = new Image();
        img.onload = () => {
            const ctx = this.$refs.canvas.getContext('2d');
            ctx.clearRect(0, 0, this.$refs.canvas.width, this.$refs.canvas.height);
            ctx.drawImage(img, 0, 0);
        };
        console.log("Loading history state", this.historyStep, this.history[this.historyStep].substring(0, 30));
        img.src = this.history[this.historyStep];
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