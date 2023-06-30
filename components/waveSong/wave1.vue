<template>
    <div>
        <div class="">
            <audio hidden autoplay="true" @loadedMetaData="loadding = true" ref="audio" :src="song" controls></audio>
            <audio hidden autoplay="true" ref="audio2" :src="song" controls></audio>
            <div class="btn btn-secondary" @click="changeSong()">chuyển bài</div>
            <div :disabled="loadding" class="btn btn-primary" @click="load()">load nè</div>
            <div class="btn btn-outline" @click="stop()">stop</div>
            <canvas class=" p-20" ref="canvas" />
        </div>
    </div>
</template>
<script setup>
const canvas = ref();
const audio = ref();
const audio2 = ref();
const list = ref([
    '/song/TimeToAttackJormungandOST-TakuIwa_4jbr2 (4).mp3',
    '/song/thuyền quyên (12).mp3',
    '/song/y2mate.com - Thuyền Quyên AM Remix  Diệu Kiên  Áo Mời Em Cài Màu Hoa Cưới Remix Hot TikTok.mp3',
    '/song/y2mate.com - UtaMacross Mirai wa Onna no Tame ni Aru Full Song  4K60fps.mp3',
])

const loadding = ref(false)

function changeSong() {
    loadding.value = false
    const random = Math.random() * 10
    const index = ~~(random % 4)
    song.value = list.value[index]
    audio2.value.play();
    audio.value.play();
}

const song = ref(list.value[0])
function stop() {
    audio2.value.pause();
    audio.value.pause();
}
function load() {
    loadding.value = false
    audio2.value.play();
    audio.value.play();
    const ctx = canvas.value.getContext("2d");
    canvas.value.width = window.innerWidth;
    canvas.value.height = window.innerHeight;

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
            this.analyser.getByteTimeDomainData(this.dataArray);
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
            const sound = micInput * 200 || 200;
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
            // context.translate(canvas.value.width / 3, canvas.value.height / 3);

            // context.rotate(this.index * 0.043);


            // context.beginPath();
            // context.bezierCurveTo(
            //     this.x / 2, this.y / 2, this.height * -0.5 - 150, this.height + 50, this.x, this.y
            // );
            // context.stroke();

            // // if (this.index > 150) {
            // //     context.beginPath()
            // //     context.arc(this.x, this.y, this.height / 2 + this.height * 0.1, this.height * 0.05, 0, Math.PI * 2)
            // //     context.stroke()
            // //     context.beginPath()
            // //     context.moveTo(this.x, this.y + 10);
            // //     context.lineTo(this.x, this.y + 10, this.height / 2)
            // //     context.stroke()
            // // }
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
            bars.push(new Bar(i * 2, 0, 1, 512, color, i));
        }
    }
    createBars();
    function animate() {
        ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);

        const samples = source.getSamples();
        const volume = source.getVolume();
        ctx.save();

        bars.forEach((bar, i) => {
            bar.update(samples[i]);
            bar.draw(ctx, 1);
        });
        ctx.restore();
        requestAnimationFrame(animate);
    }
    animate();
}
</script>
