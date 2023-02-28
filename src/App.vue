<script setup>
import WaveSurfer from "wavesurfer.js";
import spec from "wavesurfer.js/dist/plugin/wavesurfer.spectrogram.js";
import elan from "wavesurfer.js/dist/plugin/wavesurfer.elan.js";
import timeline from "wavesurfer.js/dist/plugin/wavesurfer.timeline.js";
import { onMounted } from "vue";
let wavesurfer;
onMounted(() => {
  wavesurfer = WaveSurfer.create({
    container: "#waveform",
    waveColor: [
      // an array of colors, to be applied as gradient color stops to the waveform.
      "red",
      "green",
      "purple",
      "yellow",
      "rgba(0,255,255,.5)",
    ],
    progressColor: [
      // the gradient fill styles are also available on the progressColor option
      "orange",
      "blue",
      "cyan",
      "black",
      "rgba(0,255,255,.5)",
    ],
    height: 400,
    width: 1000,
    plugins: [
      spec.create({
        wavesurfer: wavesurfer,
        container: "#wave-spectrogram",
        labels: true,
        height: 400,
      }),
      timeline.create({
        container: "#wave-timeline",
      }),
      elan.create({
        url: "http://192.168.233.1:3000",
        container: "#annotations",
        tiers: {
          鸟鸣: true,
        },
      }),
    ],
  });
  wavesurfer.load(
    "https://wavesurfer-js.org/example/elan/transcripts/001z.mp3"
  );
  wavesurfer.elan.on("select", function (start, end) {
    wavesurfer.backend.play(start, end);
  });

  let prevAnnotation, prevRow, region;
  let onProgress = function (time) {
    let annotation = wavesurfer.elan.getRenderedAnnotation(time);

    if (prevAnnotation != annotation) {
      prevAnnotation = annotation;

      region && region.remove();
      region = null;

      if (annotation) {
        // Highlight annotation table row
        let row = wavesurfer.elan.getAnnotationNode(annotation);
        prevRow && prevRow.classList.remove("success");
        prevRow = row;
        row.classList.add("success");
        let before = row.previousSibling;
        if (before) {
          wavesurfer.elan.container.scrollTop = before.offsetTop;
        }

        // Region
        region = wavesurfer.addRegion({
          start: annotation.start,
          end: annotation.end,
          resize: false,
          color: "rgba(223, 240, 216, 0.7)",
        });
      }
    }
  };
  wavesurfer.on("audioprocess", onProgress);
});
</script>

<template>
  <div>
    <div style="display: flex">
      <div class="wave">
        <div id="waveform"></div>
        <div id="wave-timeline"></div>
      </div>
      <div id="wave-spectrogram"></div>
    </div>
    <div style="margin-top: 100px; text-align: center">
      <button
        style="font-size: 30px"
        @click="
          () => {
            wavesurfer.playPause();
          }
        "
      >
        播放/暂停
      </button>
    </div>
    <div id="annotations"></div>
  </div>
</template>

<style>
.wave {
  flex: 1;
  height: 400px;
}

#wave-spectrogram {
  flex: 1;
  height: 400px;
}

#annotations {
  margin-top: 100px;
  max-height: 400px;
  overflow: auto;
}

.wavesurfer-annotations {
  border-collapse: collapse;
  width: 100%;
  border: 1px solid #c6c6c6 ;
  margin-bottom: 20px;
}

.wavesurfer-annotations th {
  border-right: 1px solid #c6c6c6 ;
  border-bottom: 1px solid #c6c6c6 ;
  background-color: #ddeeff;
  padding: 15px 19px;
  font-size: 30px;
  text-align: center;
}

.wavesurfer-annotations td {
  border-collapse: collapse;
  border-right: 1px solid #c6c6c6 ;
  border-bottom: 1px solid #c6c6c6;
  padding: 15px 19px;
  font-size: 30px;
  text-align: center;
}

.wavesurfer-annotations tr:nth-child(odd) {
  background-color: #fff;
}

.wavesurfer-annotations tr:nth-child(even) {
  background-color: #f8f8f8;
}

.success {
  background-color: #fff895 !important;
}
</style>
