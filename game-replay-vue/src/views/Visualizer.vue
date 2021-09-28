<template>
  <div class="visualizer">
    <div id="p5Canvas"></div>
  </div>
</template>

<script>
import P5 from "p5";
import game from "../output.json";

export default {
  mounted() {
    const script = (p5) => {
      // These are your typical setup() and draw() methods
      const canvasSize = 800;
      p5.setup = () => {
        const canvas = p5.createCanvas(canvasSize, canvasSize);
        canvas.parent("p5Canvas");
      };
      function mapGen(map) {
        const mSize = map.length;
        p5.noStroke();
        for (let i = 0; i < mSize; i++) {
          for (let j = 0; j < mSize; j++) {
            p5.fill(0);
            p5.square(
              (i * mSize) / canvasSize,
              (j * mSize) / canvasSize,
              mSize / canvasSize
            );
          }
        }
      }
      p5.draw = () => {
        p5.clear();
        const gr = p5.color(200, 200, 200);
        p5.noStroke();
        // mapGen(game.init.map);
        const mSize = game.init.map.length;
        for (let i = 0; i < mSize; i++) {
          for (let j = 0; j < mSize; j++) {
            p5.fill(game.init.map[i][j] * 50);
            p5.square(
              canvasSize / 8 + (i * ((canvasSize * 6) / 8)) / mSize,
              (j * ((canvasSize * 6) / 8)) / mSize,
              canvasSize / mSize
            );
          }
        }

        p5.fill(gr);
        p5.rect(0, 0, canvasSize / 8, canvasSize);
        p5.rect((canvasSize * 7) / 8, 0, canvasSize / 8, canvasSize);
        p5.rect(
          canvasSize / 8,
          (canvasSize * 6) / 8,
          (canvasSize * 6) / 8,
          canvasSize / 4
        );
        // p5.fill(0);
        // p5.textSize(32);
        // p5.text(game, canvasSize/8, canvasSize/8);
      };
    };
    // Attach the canvas to the div
    const p5canvas = new P5(script, "canvas");
  },
};
</script>

<style scoped>
#p5Canvas {
  margin: auto;
}
</style>