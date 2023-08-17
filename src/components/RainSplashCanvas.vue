<script setup lang="ts">
import { onMounted, ref } from "vue";
const props = defineProps<{ animationSpeed: number }>();
onMounted(() => {
  splashAnimation();
  adjustCanvasSize();
  window.addEventListener("resize", adjustCanvasSize);
});

let rainStop = () => {};
const adjustCanvasSize = () => {
  const canvas = canvasGroundRef.value;
  if (!canvas) return;
  const rect = canvas.getBoundingClientRect();
  canvas.width = rect.width;
  canvas.height = rect.height;
  RainSplash.init(rect.width, rect.height);
  rainStop();
  rainStop = splashAnimation();
};
interface Point {
  X: number;
  Y: number;
}

class RainSplash {
  location: Point;
  speed: Point;
  static height: number;
  static width: number;
  static get g() {
    return 120;
  }
  static get dense() {
    return 0.05;
  }
  static get RADIUS() {
    return 3;
  }
  static get MAX_SpeedY() {
    return Math.sqrt(
      2 * RainSplash.g * (RainSplash.height - 2 * RainSplash.RADIUS)
    );
  }
  static init(width: number, height: number) {
    RainSplash.height = height;
    RainSplash.width = width;
  }
  constructor() {
    this.location = {
      X: Math.random() * RainSplash.width,
      Y: RainSplash.height - RainSplash.RADIUS,
    };
    this.speed = {
      X: (Math.random() - 0.5) * RainSplash.RADIUS * 30,
      Y: Math.random() * RainSplash.MAX_SpeedY,
    };
  }
  reset() {
    this.location = {
      X: Math.random() * RainSplash.width,
      Y:
        RainSplash.RADIUS +
        Math.random() * (RainSplash.height - RainSplash.RADIUS * 2) * 0.5,
    };
    this.speed = {
      X: (Math.random() - 0.5) * RainSplash.RADIUS * 30,
      Y: 0,
    };
  }

  update(deltaTime: number) {
    this.speed.Y += RainSplash.g * (deltaTime / 1000) * props.animationSpeed;
    this.location.Y += this.speed.Y * (deltaTime / 1000) * props.animationSpeed;
    this.location.X += this.speed.X * (deltaTime / 1000) * props.animationSpeed;
    // Reset raindrop to the top if it goes out of the canvas
    if (this.location.Y + RainSplash.RADIUS > RainSplash.height) {
      if (this.speed.Y > RainSplash.MAX_SpeedY * 0.1) {
        this.speed.Y *= -0.8;
      } else {
        this.reset();
      }
    }
    if (
      this.location.X + RainSplash.RADIUS < 0 ||
      this.location.X - RainSplash.RADIUS > RainSplash.width
    ) {
      this.reset();
    }
  }
}

const canvasGroundRef = ref<HTMLCanvasElement | null>(null);
function splashAnimation() {
  let isStoped = false;
  const canvas = canvasGroundRef.value;
  if (!canvas)
    return () => {
      isStoped = true;
    };

  const rect = canvas.getBoundingClientRect();
  canvas.width = rect.width;
  canvas.height = rect.height;
  RainSplash.init(rect.width, rect.height);
  if (!canvas.getContext)
    return () => {
      isStoped = true;
    };
  const ctx = canvas.getContext("2d");
  if (!ctx)
    return () => {
      isStoped = true;
    };
  let lastTime = performance.now();
  const splashs: RainSplash[] = [];
  for (let i = 0; i < RainSplash.dense * RainSplash.width; ++i) {
    splashs.push(new RainSplash());
  }
  (function draw() {
    const now = performance.now();
    const deltaTime = now - lastTime;
    ctx.clearRect(0, 0, RainSplash.width, RainSplash.height);
    splashs.forEach((splash) => {
      splash.update(deltaTime);
      ctx.fillStyle = `rgba(255,255,255,${Math.min(
        Math.abs(splash.speed.Y / RainSplash.MAX_SpeedY) * 0.4,
        (splash.location.X / RainSplash.RADIUS - 1) * 0.02,
        ((RainSplash.width - splash.location.X) / RainSplash.RADIUS - 1) * 0.02
      )})`;
      ctx.beginPath();
      ctx.arc(
        splash.location.X,
        splash.location.Y,
        RainSplash.RADIUS,
        0,
        Math.PI * 2,
        true
      );
      ctx.fill();
    });

    lastTime = now;
    if (!isStoped) {
      requestAnimationFrame(draw);
    }
  })();
  return () => {
    isStoped = true;
  };
}
</script>

<template>
  <canvas
    ref="canvasGroundRef"
    style="height: 20px; width: 100%; margin-bottom: -4px"
  ></canvas>
</template>

<style>
body {
  margin: 0;
}
</style>
