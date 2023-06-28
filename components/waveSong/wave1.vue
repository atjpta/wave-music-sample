<template>
    <div>
        <div class="">
            <audio ref="audio" src="/song/y2mate.com - UtaMacross Mirai wa Onna no Tame ni Aru Full Song  4K60fps.mp3"
                controls></audio>
            <audio ref="audio2" src="/song/y2mate.com - UtaMacross Mirai wa Onna no Tame ni Aru Full Song  4K60fps.mp3"
                controls></audio>
            <div class="btn btn-primary" @click="load()">load nè</div>
            <div class="btn btn-outline" @click="stop = false">stop</div>
            <canvas class=" " ref="canvas" />
        </div>
    </div>
</template>
<script setup>
const canvas = ref();
const audio = ref();
const audio2 = ref();
const stop = ref(false);
function load() {
    stop.value = true;
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

    // class Bar {
    //     constructor(x, y, width, height, color, index) {
    //         this.x = x;
    //         this.y = y;
    //         this.width = width;
    //         this.height = height;
    //         this.color = color;
    //         this.index = index;
    //     }
    //     update(micInput) {
    //         const sound = micInput * 500 || 500;
    //         this.x++;
    //         if (sound > this.height) {
    //             this.height = sound;
    //         } else {
    //             this.height -= this.height * 0.03;
    //         }
    //     }
    //     draw(context, volume) {
    //         // context.fillStyle = this.color;
    //         // context.fillRect(this.x, this.y, this.width, this.height);

    //         context.strokeStyle = this.color;
    //         context.lineWidth = this.width;
    //         context.save();
    //         context.rotate(this.index * 0.05);
    //         context.beginPath();
    //         context.bezierCurveTo(
    //             this.x / 2,
    //             this.y / 2,
    //             this.height * -0.5 - 150,
    //             this.height + 50,
    //             this.x,
    //             this.y
    //         );
    //         context.stroke();

    //         context.restore();
    //     }
    // }
    class Bar {
        constructor(x, y, width, height, color, index) {
            this.x = x;
            this.y = y;
            this.width = width;
            this.height = height;
            this.color = color;
            this.index = index;
            this.angle = 0; // Góc quay của vỏ ốc sên
        }

        update(micInput) {
            const sound = micInput * 500 || 500;
            this.x++;
            if (sound > this.height) {
                this.height = sound;
            } else {
                this.height -= this.height * 0.03;
            }

            this.angle += 0.05; // Tăng góc quay sau mỗi lần cập nhật
        }

        draw(context, volume) {
            context.strokeStyle = this.color;
            context.lineWidth = this.width;
            context.save();
            context.translate(this.x, this.y); // Di chuyển tâm vẽ đến vị trí (this.x, this.y)

            const numArms = 5; // Số cánh của vỏ ốc sên
            const armLength = this.height / 2; // Độ dài của từng cánh

            context.beginPath();

            for (let i = 0; i < 360 * numArms; i++) {
                const angle = i * (Math.PI / 180);
                const radius = armLength + (angle / (Math.PI * 2)) * this.height;
                const x = radius * Math.cos(angle + this.angle);
                const y = radius * Math.sin(angle + this.angle);

                if (i === 0) {
                    context.moveTo(x, y);
                } else {
                    context.lineTo(x, y);
                }
            }

            context.stroke();
            context.restore();
        }
    }

    let fftSize = 512;
    const source = new Source(fftSize);
    let bars = [];
    let barWidth = canvas.value.width / fftSize;
    function createBars() {
        for (let i = 0; i < fftSize / 2; i++) {
            let color = `hsl(${i * 2}, 100%, 50%)`;
            bars.push(new Bar(0, i * 0.9, 1, 50, color, 1));
        }
    }
    createBars();
    function animate() {
        ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);

        const samples = source.getSamples();
        const volume = source.getVolume();
        ctx.save();
        ctx.translate(canvas.value.width / 2 - 70, canvas.value.height / 2 + 50);

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
