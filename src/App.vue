<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue";
import RainSplashCanvas from "./components/RainSplashCanvas.vue";
const canvasRef = ref<HTMLCanvasElement | null>(null);
const canvasBackgroundRef = ref<HTMLCanvasElement | null>(null);

let rainStop = () => {};
onMounted(() => {
  adjustCanvasSize();
  window.addEventListener("resize", adjustCanvasSize);
  const canvasBG = canvasBackgroundRef.value;
  if (canvasBG) {
    const ctx = canvasBG.getContext("2d");
    if (!ctx) return;
    ctx.fillStyle = "rgb(59,72,87)";
    ctx.fillRect(0, 0, canvasBG.width, canvasBG.height);
  }
  rainStop = rainAnimation();
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", adjustCanvasSize);
  rainStop();
});

const adjustCanvasSize = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const rect = canvas.getBoundingClientRect();
  canvas.width = rect.width;
  canvas.height = rect.height;
  canvasHeight.value = rect.height;

  const canvasBG = canvasBackgroundRef.value;
  if (!canvasBG) return;
  canvasBG.width = rect.width;
  canvasBG.height = rect.height;
  Raindrop.init(rect.width, rect.height);
  rainStop();
  rainStop = rainAnimation();

  const ctx = canvas.getContext("2d");
  if (!ctx) return;

  ctx.imageSmoothingEnabled = true;
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.shadowOffsetX = 0;
  ctx.shadowOffsetY = 0;
  ctx.shadowColor = "rgb(255,255,220)"; // 这会是泛光的颜色
  ctx.lineCap = "round";
  ctx.lineJoin = "round";
};

const canvasHeight = ref(0);

const animationSpeedExp = ref(0);
const animationSpeed = computed(() => Math.pow(2, animationSpeedExp.value));
const isAnimating = ref(false);
function startAnimation(e: MouseEvent) {
  if (isAnimating.value) return;
  isAnimating.value = true;
  const thunderData = new Thunder(canvasHeight.value).thunderData();
  const basePoint = { X: e.x, Y: e.y };
  const mainThunderWidth = 8;
  const branchThunderWidth = 2;

  let lastTime = performance.now();
  let percentage = 0;
  const pathAnimationTimeCost = 190;
  const lightningAnimationTimeCost = 10;
  const lightningFadeAnimationTimeCost = 600;

  pathAnimation();
  function pathAnimation() {
    const animationTime = pathAnimationTimeCost / animationSpeed.value;
    const canvas = canvasRef.value;
    if (!canvas) return;
    const now = performance.now();
    percentage += (now - lastTime) / animationTime;
    lastTime = now;
    if (canvas.getContext) {
      const ctx = canvas.getContext("2d");
      if (!ctx) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.strokeStyle = "rgb(255,255,255)";

      ctx.beginPath();
      ctx.moveTo(basePoint.X, basePoint.Y);
      const drawThunder = (thunderPath: Point[], end: number) => {
        ctx.moveTo(
          basePoint.X + thunderPath[0].X,
          basePoint.Y + thunderPath[0].Y
        );
        thunderPath.forEach((pt, index) => {
          if (index > end) return;
          ctx.lineTo(basePoint.X + pt.X, basePoint.Y + pt.Y);
        });
        ctx.stroke();
      };
      ctx.lineWidth = branchThunderWidth;
      ctx.shadowBlur = 0; // 调整模糊量以增加/减少泛光的范围
      const endIndex = thunderData.main.length * percentage;
      drawThunder(thunderData.main, endIndex);
      thunderData.branch.forEach((branch, index) => {
        drawThunder(branch, endIndex - thunderData.branchIndex[index]);
      });
    }
    if (percentage < 1) {
      requestAnimationFrame(pathAnimation);
    } else {
      percentage = 0;
      lightningAnimation();
    }
  }
  function lightningAnimation() {
    const animationTime = lightningAnimationTimeCost / animationSpeed.value;

    const canvas = canvasRef.value;
    if (!canvas) return;
    const now = performance.now();
    percentage += (now - lastTime) / animationTime;
    lastTime = now;
    if (canvas.getContext) {
      const ctx = canvas.getContext("2d");
      if (!ctx) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.shadowBlur = 0;
      ctx.fillStyle = `rgba(255,255,255,${percentage / 2})`;
      ctx.strokeStyle = "rgb(255,255,255)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      ctx.beginPath();
      ctx.moveTo(basePoint.X, basePoint.Y);
      const drawThunder = (thunderPath: Point[]) => {
        ctx.moveTo(
          basePoint.X + thunderPath[0].X,
          basePoint.Y + thunderPath[0].Y
        );
        thunderPath.forEach((pt) => {
          ctx.lineTo(basePoint.X + pt.X, basePoint.Y + pt.Y);
        });
        ctx.stroke();
      };
      ctx.lineWidth =
        branchThunderWidth +
        (mainThunderWidth - branchThunderWidth) * percentage;

      ctx.shadowBlur = ctx.lineWidth * 4; // 调整模糊量以增加/减少泛光的范围
      drawThunder(thunderData.main);
      ctx.lineWidth = branchThunderWidth;
      ctx.shadowBlur = 0; // 调整模糊量以增加/减少泛光的范围
      thunderData.branch.forEach((branch) => {
        drawThunder(branch);
      });
    }
    if (percentage < 1) {
      requestAnimationFrame(lightningAnimation);
    } else {
      percentage = 0;
      lightningFadeAnimation();
    }
  }
  function lightningFadeAnimation() {
    const animationTime = lightningFadeAnimationTimeCost / animationSpeed.value;

    const canvas = canvasRef.value;
    if (!canvas) return;
    const now = performance.now();
    percentage += (now - lastTime) / animationTime;
    lastTime = now;
    if (canvas.getContext) {
      const ctx = canvas.getContext("2d");
      if (!ctx) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.shadowBlur = 0;
      ctx.fillStyle = `rgba(255,255,255,${(1 - percentage) / 2})`;
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      ctx.beginPath();
      ctx.moveTo(basePoint.X, basePoint.Y);
      const drawThunder = (thunderPath: Point[]) => {
        ctx.moveTo(
          basePoint.X + thunderPath[0].X,
          basePoint.Y + thunderPath[0].Y
        );
        thunderPath.forEach((pt) => {
          ctx.lineTo(basePoint.X + pt.X, basePoint.Y + pt.Y);
        });
        ctx.stroke();
      };
      ctx.lineWidth = mainThunderWidth * (1 - percentage);
      ctx.shadowBlur = ctx.lineWidth * 2; // 调整模糊量以增加/减少泛光的范围
      ctx.strokeStyle = `rgba(255,255,225,${Math.pow(1 - percentage, 0.3)})`;
      drawThunder(thunderData.main);
      if (branchThunderWidth - (mainThunderWidth / 2) * percentage > 0) {
        ctx.lineWidth =
          branchThunderWidth - (mainThunderWidth / 2) * percentage;
        ctx.shadowBlur = 0; // 调整模糊量以增加/减少泛光的范围
        thunderData.branch.forEach((branch) => {
          drawThunder(branch);
        });
      }
      if (percentage < 1) {
        requestAnimationFrame(lightningFadeAnimation);
      } else {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        isAnimating.value = false;
      }
    }
  }
}

interface Point {
  X: number;
  Y: number;
}

class Thunder {
  #main: Point[];
  #branch: Point[][] = [];
  #branchIndex: number[] = [];
  static get RADIUS() {
    return 4;
  }

  constructor(minLength: number) {
    const getRangeNormalRandom = (() => {
      function getRandom() {
        return Math.random();
      }

      function boxMullerTransform() {
        let u = 0;
        let v = 0;

        // 保证 u 和 v 都不为0
        while (u === 0) u = getRandom();
        while (v === 0) v = getRandom();

        // 标准正态分布
        return Math.sqrt(-2.0 * Math.log(u)) * Math.cos(2.0 * Math.PI * v);
      }

      function getNormalRandom(mean: number, stdDev: number) {
        return mean + stdDev * boxMullerTransform();
      }

      return (mean: number, stdDev: number, min: number, max: number) => {
        let ans = getNormalRandom(mean, stdDev);
        while (ans > max || ans < min) ans = getNormalRandom(mean, stdDev);
        return ans;
      };
    })();
    this.#main = [{ X: 0, Y: 0 }];
    function getThunderPath(
      baseAngle: number,
      lastPoint = { X: 0, Y: 0 },
      length?: number
    ) {
      const main = [lastPoint];
      let lastRandNumber = baseAngle + Math.PI / 2;
      function nextPoint() {
        let randNumber =
          baseAngle +
          getRangeNormalRandom(Math.PI / 2, 1, 0.25, Math.PI - 0.15);
        const maxDelta = 0.5;

        if (Math.abs(lastRandNumber - randNumber) > maxDelta) {
          randNumber =
            lastRandNumber +
            (lastRandNumber > randNumber ? -maxDelta : maxDelta);
        }
        const currentPoint: Point = {
          X: lastPoint.X + Thunder.RADIUS * Math.cos(randNumber),
          Y: lastPoint.Y - Thunder.RADIUS * Math.sin(randNumber),
        };
        main.push(currentPoint);
        lastPoint = currentPoint;
        lastRandNumber = randNumber;
      }
      if (length) {
        for (let i = 0; i < length; ++i) {
          nextPoint();
        }
      } else {
        while (main[main.length - 1].Y + minLength > 0) {
          nextPoint();
        }
      }
      return main;
    }
    const mainAngle = getRangeNormalRandom(0, 0.25, -0.5, 0.5);
    this.#main = getThunderPath(mainAngle).reverse();
    const length = this.#main.length;
    for (let i = 0; i < 3; ++i) {
      this.#branchIndex.push(
        Math.floor(
          getRangeNormalRandom(
            length * 0.6,
            length / 6,
            length * 0.3,
            length * 0.9
          )
        )
      );
    }
    this.#branchIndex.sort().forEach((index) => {
      const sideTunderPath = getThunderPath(
        mainAngle +
          Math.PI +
          getRangeNormalRandom(Math.PI * 0.25, 0.25, 0, Math.PI / 2) *
            (Math.random() > 0.5 ? -1 : 1),
        this.#main[index],
        (length - index) / 1.5
      );
      this.#branch.push(sideTunderPath);
    });
  }

  thunderData(percent: number = 0) {
    return {
      main: this.#main,
      branch: this.#branch,
      branchIndex: this.#branchIndex,
    };
  }
}

class Raindrop {
  location: Point;
  distance: number;
  static height: number;
  static width: number;
  static get MAX_Distance() {
    return 0.8;
  }
  static get SPEED() {
    return 800 * animationSpeed.value;
  }
  static get LEN() {
    return 24;
  }
  static init(width: number, height: number) {
    Raindrop.height = height;
    Raindrop.width = width;
  }
  constructor() {
    this.location = {
      X: Math.random() * Raindrop.width,
      Y: 2 * Math.random() * Raindrop.height,
    };
    this.distance = Math.random() * Raindrop.MAX_Distance;
  }

  get speed() {
    return Raindrop.SPEED * (1 - this.distance);
  }

  get length() {
    return Raindrop.LEN * (1 - this.distance);
  }

  update(deltaTime: number) {
    this.location.Y += this.speed * (deltaTime / 1000);
    // Reset raindrop to the top if it goes out of the canvas
    if (this.location.Y - this.length > Raindrop.height) {
      this.location.Y = -Math.random() * Raindrop.height;
      this.location.X = Math.random() * Raindrop.width;
    }
  }
}

function rainAnimation() {
  let isStoped = false;
  const canvas = canvasBackgroundRef.value;
  if (!canvas)
    return () => {
      isStoped = true;
    };
  const rect = canvas.getBoundingClientRect();
  Raindrop.init(rect.width, rect.height);
  const ctx = canvas.getContext("2d");
  if (!ctx)
    return () => {
      isStoped = true;
    };
  const dense = 12;
  const count = (rect.width / 100) * (rect.height / 100) * dense;
  const raindrops: Raindrop[] = [];
  for (let i = 0; i < count; ++i) {
    raindrops.push(new Raindrop());
  }
  let lastTime = performance.now();
  (function animate() {
    const now = performance.now();
    ctx.fillStyle = "rgb(59,72,87)";
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    for (const drop of raindrops) {
      drop.update(now - lastTime);
      ctx.beginPath();

      //圆头尖尾雨滴
      // ctx.fillStyle = `rgba(255,255,255,${(1 - drop.distance) * 0.8})`;
      // ctx.moveTo(drop.location.X, drop.location.Y - drop.length);
      // ctx.arc(drop.location.X, drop.location.Y, drop.length * 0.05, 0, Math.PI);
      // ctx.lineTo(drop.location.X, drop.location.Y - drop.length);
      // ctx.fill();

      //线状雨滴
      if (drop.location.Y > 0) {
        ctx.strokeStyle = `rgba(255,255,255,${(1 - drop.distance) * 0.3})`;
        ctx.lineWidth = (1 - drop.distance) * 2;
        ctx.moveTo(drop.location.X, drop.location.Y - drop.length);
        ctx.lineTo(drop.location.X, drop.location.Y);
        ctx.stroke();
      }

      //模糊雨滴 webkit 有严重性能问题
      // ctx.shadowOffsetX = 0;
      // ctx.shadowOffsetY = 0;
      // ctx.shadowBlur = 1;
      // ctx.strokeStyle = `rgba(255,255,255,${(1 - drop.distance) * 0.3})`;
      // ctx.shadowColor = `rgba(255,255,255,${1 - drop.distance})`;
      // ctx.shadowBlur = (drop.distance - 0.4) * 10;
      // ctx.moveTo(drop.location.X, drop.location.Y - drop.length);
      // ctx.lineTo(drop.location.X, drop.location.Y);
      // ctx.stroke();
    }
    lastTime = performance.now();
    if (!isStoped) {
      requestAnimationFrame(animate);
    }
  })();
  return () => {
    isStoped = true;
  };
}

const isChrome = computed(() => {
  const userAgent = navigator.userAgent;
  return /Chrome\//.test(userAgent) && !/Chromium\//.test(userAgent);
});
</script>

<template>
  <div style="position: relative; width: 100vw; height: 100vh">
    <canvas
      ref="canvasBackgroundRef"
      style="position: absolute; width: 100%; height: 100%"
    ></canvas>
    <canvas
      class="thor"
      ref="canvasRef"
      style="position: absolute; width: 100%; height: 100%"
      @click="startAnimation"
    ></canvas>

    <div style="position: absolute; bottom: 1rem; left: 1rem">
      <div style="width: 100%">
        <RainSplashCanvas :animation-speed="animationSpeed" />
      </div>
      <div
        style="
          color: white;
          backdrop-filter: blur(4px);
          background-color: rgba(0, 0, 0, 0.25);
          padding: 0.75rem 1rem;
          margin: 0;
          border-radius: 1rem;
          border: 1px solid #627388;
        "
      >
        Animation Speed: x{{
          animationSpeed >= 1
            ? animationSpeed.toFixed(1).toString()
            : `1/${(1 / animationSpeed).toFixed(1)}`
        }}<br />
        1/16
        <input
          v-model="animationSpeedExp"
          type="range"
          id="volume"
          min="-4"
          max="1"
          step="0.1"
        />
        2
        <button @click="animationSpeedExp = 0">reset</button>
        <br />
        <a
          href="https://github.com/Dar-Xs/Real-Time-Thunder-Demo"
          target="_blank"
        >
          <img
            alt="GitHub Repo stars"
            src="https://img.shields.io/github/stars/Dar-Xs/Real-Time-Thunder-Demo?logo=github"
        /></a>

        <div v-if="!isChrome">
          动画有点奇怪？<br />使用
          <a
            style="color: white"
            href="https://www.google.com/intl/zh-CN/chrome/"
            target="_blank"
            >Chrome 浏览器</a
          >，渲染效果更佳
        </div>
      </div>
    </div>
    <div style="position: absolute; bottom: 0; width: 100%">
      <RainSplashCanvas :animation-speed="animationSpeed" />
    </div>
  </div>
</template>

<style>
body {
  margin: 0;
}
.thor {
  cursor: url("/thor.png") 24 16, pointer;
}
</style>
