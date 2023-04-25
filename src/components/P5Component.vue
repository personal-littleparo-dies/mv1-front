<template>
  <div ref="canvas" />
  <input type="file" id="audiofile" @change="loadSong">
  <audio ref="audioElement"></audio>
</template>

<script lang="ts">
import p5 from "p5";
import "p5/lib/addons/p5.sound"; // see https://github.com/processing/p5.js-sound/issues/453#issuecomment-790574496

import { onMounted, ref } from 'vue';

export default {
  name: 'P5MusicVisualizer',
  // props: {
  //   audio: {
  //     type: String,
  //     required: true
  //   }
  // },
  setup() {
    const canvas = ref<HTMLDivElement | null>(null);
    const fft = new p5.FFT(0.9, 128);

    const audio = document.getElementById("audio");
    let song = ref(null)
    let audioElement = ref(null)
    let button = ref(null)

    function loadSong(event) {
      if (event.target.files[0]) {
        if (song.value) { // Catch already playing songs
          song.value.disconnect();
          song.value.stop();
        }

        // Load our new song
        audioElement.value.src = URL.createObjectURL(event.target.files[0]);
        song.value = loadSound(audioElement.value.src);
      }
    }
    function togglePlaying() {
      if (!song.isPlaying()) {
        song.play();
        song.setVolume(0.5);
        button.html('pause');
      }
      else {
        song.pause();
        button.html('play');
      }
    }

    const sketch = (p: p5) => {
      const { color, clear, noStroke, ellipse, max, translate, rotate, rect, pop, fill, push, map, min, constrain } = p;

      p.setup = () => {
        const { createCanvas, colorMode, angleMode, createButton, HSB, DEGREES } = p;

        createCanvas(800, 800);
        // colorMode(RGB);
        colorMode(HSB, 360, 100, 100)
        angleMode(DEGREES);

        const button = createButton("play");
        button.position(100, 10);
        button.size(60, 25);
        button.mousePressed(togglePlaying);
        // mic = new p5.AudioIn(); // 현재는 마이크를 입력으로 사용하고 있으나 파일을 사용하려면 따로 만들어야 한다.
        // mic.start();
        // fft.setInput(mic);
      };

      p.draw = () => {
        function getNewSoundDataValue(freqType: any) {
          return map(fft.getEnergy(freqType), 0, 255, 0, 1); // get energy from frequency, scaled from 0 to 1
        }
        function getDataHSBColor(d: number) {
          fft.dataHue = map(d, 0, 1, 0, 360); // value moves across full 360 degrees of hue range
          this.dataSaturation =
            this.dataBrightness = map(d, 0, 1, 0, 100); // higher data value = brighter, saturated color
          return color(this.dataHue, this.dataSaturation, this.dataBrightness);
        }
        // p5 draw code here
        // background(50);

        clear();
        if (song && song.isLoaded() && !song.isPlaying()) { // Do once
          // loader.classList.remove("loading");

          // song.play();
          // song.setVolume(0.4);
          fft.setInput(song);
          // fft.setInput(audio)
        }
        if (fft) {
          noStroke(); // no 태두리
          const size = 100; // 바 시각화 사이즈

          const spectrum = fft.analyze(1024); // spectrum array
          // console.log(spectrum);
          const dataVal = getNewSoundDataValue("bass"); // look bottom (function)
          // let dataVal = getNewSoundDataValue("lowMid");


          const dataColor = getDataHSBColor(dataVal); // look bottom (function)

          fill(dataColor); // 위의 실시간 색

          const myEllipseSize = 180 * dataVal // 가운데 원 사이즈
          ellipse(200, 200, myEllipseSize, myEllipseSize); // 바로 위의 원 그리기

          // const maxData = max(spectrum); // 다음에 쓸 배열 하나당 최댓값
          const angleSeparation = 360 / spectrum.length; // 360도를 배열의 갯수만큼 나눔 공간 만듬

          for (let i = 0; i < spectrum.length; i++) {
            push(); // 현재상태 저장
            const maxValue = constrain(spectrum[i], 100, 1096);
            // let animatedHeight = map(maxValue, min(spectrum), 800, 1, spectrum.reverse()[i]);
            const animatedHeight = map(maxValue, min(spectrum), 800, 1, spectrum[i]);
            translate(200, 200); // 이동(위치 바꿔야 될듯)
            rotate(angleSeparation * i); // 갯수만큼 회전시켜 완전 원 만듬
            rect(0, size, angleSeparation * 1.25, animatedHeight); // 네모시각화 최종 사이즈
            pop(); // 다시 전상태 복원
          }
        }
        /*
    
        map(value, start1, stop1, start2, stop2, [withinBounds])
      parameters
        value Number: the incoming value to be converted
        start1 Number: lower bound of the value's current range
        stop1 Number: upper bound of the value's current range
        start2 Number: lower bound of the value's target range
        stop2 Number: upper bound of the value's target range
        withinBounds Boolean: constrain the value to the newly mapped range (Optional)
    
        */
      };
    };

    onMounted(() => {
      if (canvas.value) new p5(sketch, canvas.value);
    });
    return { canvas, loadSong };
  },
}
</script>

<style scoped>
/* body {
	margin: 0;
	overflow: hidden;
	color: white;
	}
	input {
	position: absolute;
	top: 15px;
	left: 15px;
	}
  .loader {
	font-family: sans-serif;
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50% -50%);
	display: none;
	}
	.loader.loading  { display: block; }
	canvas {
		border: 1px solid black;
	}
	#audio {
		margin-top: 100px;
	}
	h3 {
		position: fixed;
		top: 5vh;
		bottom: 80vh;
		right: 10vw;
		color: #231251;
	} */
</style>
