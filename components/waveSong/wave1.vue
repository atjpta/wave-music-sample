<template>
  <div>
    <div class="overflow-hidden">
      <div>
        <audio
          loop="true"
          hidden
          autoplay="true"
          @loadedMetaData="loading = true"
          ref="audio"
          :src="song"
          controls
        ></audio>
        <audio
          loop="true"
          hidden
          autoplay="true"
          ref="audio2"
          :src="song"
          controls
        ></audio>
        <div class="btn btn-secondary" @click="changeSong()">chuyển bài</div>
        <div :disabled="loading" class="btn btn-primary" @click="load()">load nè</div>
        <div class="btn btn-outline" @click="stop()">stop</div>
      </div>
      <div class="">
        <div class="flex justify-center items-center relative">
          <!-- <span class="indicator-item indicator-center indicator-middle"> -->
          <div class="z-20 absolute">
            <img
              hidden
              ref="image"
              class="rounded-full w-32 h-32"
              src="image/girl.jpeg"
              alt=""
            />
          </div>
          <!-- </span> -->
          <div class="" id="imageSpin">
            <canvas class="" ref="canvas" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
const canvas = ref();
const audio = ref();
const audio2 = ref();
const list = ref([
  "/song/TimeToAttackJormungandOST-TakuIwa_4jbr2 (4).mp3",
  "/song/thuyền quyên (12).mp3",
  "/song/y2mate.com - Thuyền Quyên AM Remix  Diệu Kiên  Áo Mời Em Cài Màu Hoa Cưới Remix Hot TikTok.mp3",
  "/song/y2mate.com - UtaMacross Mirai wa Onna no Tame ni Aru Full Song  4K60fps.mp3",
]);
const image = ref();

const loading = ref(false);

function changeSong() {
  loading.value = false;
  const random = Math.random() * 10;
  const index = ~~(random % 4);
  song.value = list.value[index];
  audio2.value.play();
  audio.value.play();
}

const song = ref(list.value[0]);
function stop() {
  audio2.value.pause();
  audio.value.pause();
}
function load() {
  loading.value = false;
  image.value.hidden = false;
  audio2.value.play();
  audio.value.play();
  const ctx = canvas.value.getContext("2d");
  canvas.value.width = window.innerWidth - 50;
  canvas.value.height = window.innerHeight - 50;
  // // canvas.value.width = 500;
  // // canvas.value.height = 500;
  class Source {
    constructor(fftSize) {
      this.initialized = false;
      window.AudioContext = window.AudioContext || window.webkitURL.AudioContext;
      this.audioContext = new window.AudioContext();
      this.analyser = this.audioContext.createAnalyser();
      try {
        this.source = this.audioContext.createMediaElementSource(audio.value);
      } catch (error) {
        console.log(error);
      }

      // console.log(this.analyser);
      this.analyser.fftSize = fftSize;
      const bufferLength = this.analyser.frequencyBinCount;
      this.dataArray = new Uint8Array(bufferLength);
      this.source.connect(this.analyser);
      this.initialized = true;
    }
    getSamples() {
      this.analyser.getByteFrequencyData(this.dataArray);
      let normSamples = [...this.dataArray].map((e) => e / 128 - 1);
      return normSamples;
    }
    getVolume() {
      this.analyser.getByteFrequencyData(this.dataArray);
      let normSamples = [...this.dataArray].map((e) => e / 128 - 1);
      let sum = 0;
      for (let i = 0; i < normSamples.length; i++) {
        sum += normSamples[i] * normSamples[i];
      }
      let volume = Math.sqrt(sum / normSamples.length);
      return volume;
    }
  }

  class Bar {
    constructor(x, y, width, height, color, index) {
      this.x = x;
      this.y = y;
      this.width = width;
      this.height = height;
      this.color = color;
      this.index = index;
    }
    update(micInput) {
      const sound = micInput * 500 || 500;
      if (sound > this.height) {
        this.height = sound;
      } else {
        this.height -= this.height * 0.03;
      }
    }
    draw(context, volume) {
      context.fillStyle = this.color;
      context.fillRect(this.x, this.y, this.width, this.height);

      // context.strokeStyle = this.color;
      // context.lineWidth = this.width;
      // context.save();
      // context.rotate(this.index * 0.043);
      // context.beginPath();
      // context.bezierCurveTo(
      //   this.x / 2,
      //   this.y / 2,
      //   this.height * -0.5 - 150,
      //   this.height + 50,
      //   this.x,
      //   this.y
      // );
      // context.stroke();
      // if (this.index > 150) {
      //   context.beginPath();
      //   context.arc(
      //     this.x,
      //     this.y,
      //     this.height / 2 + this.height * 0.1,
      //     this.height * 0.05,
      //     0,
      //     Math.PI * 2
      //   );
      //   context.stroke();
      //   context.beginPath();
      //   context.moveTo(this.x, this.y + 10);
      //   context.lineTo(this.x, this.y + 10, this.height / 2);
      //   context.stroke();
      // }
      // context.restore();
    }
  }

  let fftSize = 1024;
  const source = new Source(fftSize);
  let bars = [];
  let barWidth = canvas.value.width / fftSize;
  function createBars() {
    for (let i = 5; i < fftSize / 4; i++) {
      let color = `hsl(${i}, 100%, 50%)`;
      bars.push(new Bar(i, 0, 5, 512, color, i));
    }
  }

  createBars();
  function animate() {
    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);

    // const samples = source.getSamples();
    // const volume = source.getVolume();

    // ctx.save();
    // ctx.translate(canvas.value.width / 2, canvas.value.height / 2);
    // bars.forEach((bar, i) => {
    //   bar.update(samples[i]);
    //   bar.draw(ctx, 1);
    // });
    // ctx.restore();
    const samples = source.getSamples();
    const volume = source.getVolume();
    drawVisualiner(fftSize / 2, 0, barWidth, 10, source.dataArray);
    // ctx.save();
    // ctx.translate(canvas.value.width / 2, canvas.value.height / 2);
    // // Bán kính của hình tròn
    // var radius = 100 / volume;

    // // Tính toán kích thước ảnh
    // var width = 2 * radius;
    // var height = 2 * radius;

    // // Đặt lại điểm gốc để nằm giữa hình tròn
    // var x = 10 - radius; // Tọa độ x của điểm gốc
    // var y = 10 - radius; // Tọa độ y của điểm gốc

    // // Vẽ hình tròn
    // ctx.beginPath();
    // ctx.arc(radius + x, radius + y, radius, 0, Math.PI * 2);
    // ctx.closePath();
    // ctx.clip();

    // // ctx.transform(volume, 0, 0, volume, 0, 0);
    // // Vẽ ảnh trong hình tròn
    // ctx.drawImage(image.value, x, y, width, height);

    // ctx.restore();
    const vScale = (10 * volume) / 3;
    image.value.style.transform = `scale(${vScale})`;
    requestAnimationFrame(animate);
  }
  animate();

  function drawVisualiner(bufferLength, x, barWidth, barheight, dataArray) {
    for (let i = 0; i < bufferLength; i++) {
      const barheight = dataArray[i] * 2;

      ctx.save();
      ctx.translate(canvas.value.width / 2, canvas.value.height / 2);

      // ctx.rotate(i * 4.184);
      ctx.rotate(i);
      const heu = 120 + i * 0.7;
      ctx.fillStyle = `hsl(${heu}, 100%, 50%)`;
      ctx.beginPath();
      ctx.arc(50, barheight / 2, barheight / 2, 0, Math.PI / 4);
      ctx.fill();
      ctx.stroke();
      x += barWidth;
      ctx.restore();
    }
  }
}
</script>

<style>
.wrapper-main {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

#imageSpin {
  animation: rotate 5s linear infinite;
}

#canvas {
  position: absolute;
  z-index: 9;
  width: 500px;
  height: 500px;
}

.containerCanvas {
  position: relative;
  width: 500px;
  height: 500px;
  display: flex;
  justify-content: center;
  align-items: center;
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}
</style>
