<template>
  <div id="app">
    <div v-if="!isLoaded">
      <label>
        Upload your replay JSON:
        <input
          type="file"
          id="replay"
          ref="file"
          accept="application/json"
          v-on:change="handleFileUpload()"
        />
      </label>
    </div>
    <div id="p5Canvas"></div>
  </div>
</template>

<script>
import P5 from "p5";
import game from "./output.json";

export default {
  data() {
    return {
      isLoaded: false,
      mapLoad: false,
    };
  },
  methods: {
    handleFileUpload() {
      const file = this.$refs.file.files[0];
      const reader = new FileReader();
      let y;
      reader.onload = (event) => {
        y = event.target.result;
        this.replay(JSON.parse(y));
      };
      reader.readAsText(file);
      this.isLoaded = true;
    },
    replay(game) {
      console.log(game);
      const script = (p5) => {
        const canvasSize = window.innerHeight - 20;
        function mapGen(map, b, p5) {
          console.log("mapping");
          const { fill, square } = p5;
          const mSize = map.length;
          for (let i = 0; i < mSize; i++) {
            for (let j = 0; j < mSize; j++) {
              switch (map[i][j]) {
                case 0:
                  b.fill(0, 200, 0);
                  break;
                case 1:
                  b.fill(100);
                  break;
                case 2:
                  b.fill(0);
                  break;
                case 3:
                  b.fill("grey");
                  break;
                case 4:
                  b.fill("brown");
                  break;
                default:
                  b.fill(0);
                  break;
              }
              b.noStroke();
              b.square(
                (i * ((canvasSize * 6) / 8)) / mSize,
                (j * ((canvasSize * 6) / 8)) / mSize,
                canvasSize / mSize
              );
            }
          }
        }
        let bg;
        p5.setup = () => {
          const canvas = p5.createCanvas(canvasSize, canvasSize);
          bg = p5.createGraphics((canvasSize * 6) / 8, (canvasSize * 6) / 8);
          mapGen(game.init.map, bg, p5);
          canvas.parent("p5Canvas");
          p5.frameRate(30);
        };
        function drawUi() {
          p5.noStroke();
          const gr = p5.color(200, 200, 200);
          p5.fill(gr);
          p5.rect(0, 0, canvasSize / 8, canvasSize);
          p5.rect((canvasSize * 7) / 8, 0, canvasSize / 8, canvasSize);
          p5.rect(
            canvasSize / 8,
            (canvasSize * 6) / 8,
            (canvasSize * 6) / 8,
            canvasSize / 4
          );
          p5.fill(0);
          p5.textSize(32);
        }

        function doPhase(turn, text, mSize) {
          p5.text(text, canvasSize / 2, 40 + (canvasSize * 7) / 8);
          let p1u = 0;
          let p2u = 0;
          let p1A = 0;
          let p2A = 0;
          let p1G = 0;
          let p2G = 0;
          for (let i = 0; i < turn.units.length; i++) {
            const unit = turn.units[i];
            let col = "";
            if (unit.owner == 0) {
              col = "blue";
              p1u++;
            } else {
              col = "red";
              p2u++;
            }
            p5.strokeWeight(3);
            p5.stroke(col);
            let fil = "";
            switch (unit.type) {
              case "gatherer":
                fil = "yellow";
                if (unit.owner == 0) {
                  p1G++;
                } else {
                  p2G++;
                }
                break;
              case "attacker":
                fil = "purple";
                if (unit.owner == 0) {
                  p1A++;
                } else {
                  p2A++;
                }
                break;
            }
            p5.fill(fil);
            const x =
              (unit.position[0] * ((canvasSize * 6) / 8)) / mSize +
              canvasSize / 8 +
              (canvasSize * 6) / 8 / (2 * mSize);
            const y =
              (unit.position[1] * ((canvasSize * 6) / 8)) / mSize +
              (canvasSize * 6) / 8 / (2 * mSize);
            p5.circle(x, y, 10);
          }
          p5.textSize(18);
          p5.fill(50);
          p5.strokeWeight(1);
          p5.stroke("Blue");
          p5.text("Player 1", canvasSize / 16, 40);
          p5.text("Units: " + p1u, canvasSize / 16, 70);
          p5.text("Gatherers: " + p1G, canvasSize / 16, 90);
          p5.text("Attackers: " + p1A, canvasSize / 16, 110);
          p5.text("Wood: " + turn.players[0].wood, canvasSize / 16, 140);
          p5.text("Metal: " + turn.players[0].metal, canvasSize / 16, 160);

          p5.stroke("Red");
          p5.text("Player 2", (15 * canvasSize) / 16, 40);
          p5.text("Units: " + p2u, (15 * canvasSize) / 16, 70);
          p5.text("Gatherers: " + p2G, (15 * canvasSize) / 16, 90);
          p5.text("Attackers: " + p2A, (15 * canvasSize) / 16, 110);
          p5.text("Wood: " + turn.players[1].wood, (15 * canvasSize) / 16, 140);
          p5.text(
            "Metal: " + turn.players[1].metal,
            (15 * canvasSize) / 16,
            160
          );
        }

        let update = false;
        let turn = 0;
        let phase = 0;
        let ended = false;
        p5.draw = () => {
          p5.textSize(32);
          p5.textAlign(p5.CENTER);
          if (!update) {
            p5.clear();
            p5.image(bg, canvasSize / 8, 0);
            console.log("updated");
            update = true;
          }

          if (p5.frameCount % 30 == 1 && !ended) {
            p5.image(bg, canvasSize / 8, 0);
            drawUi();
            p5.text(turn + 1, canvasSize / 2, (canvasSize * 7) / 8);
            let tcenter = "";
            let attribute = game.turns[turn];
            switch (phase) {
              case 0:
                tcenter = "Attack";
                attribute = game.turns[turn].attack;
                break;
              case 1:
                tcenter = "Move";
                attribute = game.turns[turn].move;
                break;
              case 2:
                tcenter = "Collect";
                attribute = game.turns[turn].collect;
                break;
              case 3:
                tcenter = "Spawn";
                attribute = game.turns[turn].spawn;
                break;
            }
            doPhase(attribute, tcenter, game.init.map.length);
            phase++;
            if (phase == 4) {
              turn++;
              phase = 0;
              if (turn == game.turns.length) {
                ended = true;
                p5.noStroke();
                p5.fill(0);
                p5.text("Game Over", canvasSize / 2, 80 + (canvasSize * 7) / 8);
              }
            }
          }
        };
      };
      const p5canvas = new P5(script, "canvas");
    },
  },
};
</script>

<style>
#p5Canvas {
  margin: auto;
}
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}
</style>
