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
        const canvasSizex = canvasSize + canvasSize / 4;
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
          const canvas = p5.createCanvas(canvasSizex, canvasSize);
          bg = p5.createGraphics((canvasSize * 6) / 8, (canvasSize * 6) / 8);
          mapGen(game.init.map, bg, p5);
          canvas.parent("p5Canvas");
          p5.frameRate(30);
        };

        function drawUi(turn, leng, pause, sides) {
          p5.noStroke();
          if (sides) {
            const gr = p5.color(200, 200, 200);
            p5.fill(gr);
            p5.rect(0, 0, (2 * canvasSizex) / 10, canvasSize);
            p5.rect(
              (8 * canvasSizex) / 10,
              0,
              (2 * canvasSizex) / 10,
              canvasSize
            );
            p5.rect(
              (2 * canvasSizex) / 10,
              (canvasSize * 6) / 8,
              (6 * canvasSizex) / 10,
              canvasSize / 4
            );
            p5.fill(150, 150, 150);
            p5.rect(
              (canvasSizex * 2) / 10,
              (canvasSize * 7) / 8,
              (canvasSizex * 6) / 10,
              10
            );

            p5.fill(50, 50, 255);
            p5.rect(
              (canvasSizex * 2) / 10,
              (canvasSize * 7) / 8,
              (turn / leng) * ((canvasSizex * 6) / 10),
              10
            );
          }
          p5.fill(0);
          p5.textSize(32);

          p5.fill(0, 50, 0);
          p5.rect(canvasSizex / 2 - 50, (canvasSize * 12) / 16 + 10, 100, 40);
          p5.rect(canvasSizex / 2 - 200, (canvasSize * 12) / 16 + 10, 100, 40);
          p5.rect(canvasSizex / 2 + 100, (canvasSize * 12) / 16 + 10, 100, 40);

          p5.fill(255);
          p5.text("Back", canvasSizex / 2 - 150, (canvasSize * 12) / 16 + 40);
          p5.text("Next", canvasSizex / 2 + 150, (canvasSize * 12) / 16 + 40);
          if (pause) {
            p5.text("Play", canvasSizex / 2, (canvasSize * 12) / 16 + 40);
          } else {
            p5.text("Pause", canvasSizex / 2, (canvasSize * 12) / 16 + 40);
          }
        }

        function drawUnit(unit, x, y) {
          let col = "";
          if (unit.owner == 0) {
            col = "blue";
          } else {
            col = "red";
          }
          p5.strokeWeight(3);
          p5.stroke(col);
          let fil = "";
          switch (unit.type) {
            case "gatherer":
              fil = "yellow";

              break;
            case "attacker":
              fil = "purple";
              break;
          }
          p5.fill(fil);
          p5.circle(x, y, 10);
        }

        function doPhase(turn, mSize) {
          p5.fill(0);
          let p1u = 0;
          let p2u = 0;
          let p1A = 0;
          let p2A = 0;
          let p1G = 0;
          let p2G = 0;
          for (let i = 0; i < turn.units.length; i++) {
            const unit = turn.units[i];
            if (unit.health > 0) {
              if (unit.owner == 0) {
                p1u++;
              } else {
                p2u++;
              }
              switch (unit.type) {
                case "gatherer":
                  if (unit.owner == 0) {
                    p1G++;
                  } else {
                    p2G++;
                  }
                  break;
                case "attacker":
                  if (unit.owner == 0) {
                    p1A++;
                  } else {
                    p2A++;
                  }
                  break;
              }
            }
          }
          p5.textSize(18);
          p5.fill(50);
          p5.strokeWeight(1);
          p5.stroke("Blue");
          p5.text("Player 1", canvasSizex / 10, 40);
          p5.text("Units: " + p1u, canvasSizex / 10, 70);

          p5.stroke("Red");
          p5.text("Player 2", (9 * canvasSizex) / 10, 40);
          p5.text("Units: " + p2u, (9 * canvasSizex) / 10, 70);

          p5.stroke("yellow");
          p5.text("Gatherers: " + p1G, canvasSizex / 10, 90);
          p5.text("Gatherers: " + p2G, (9 * canvasSizex) / 10, 90);

          p5.stroke("purple");
          p5.text("Attackers: " + p1A, canvasSizex / 10, 110);
          p5.text("Attackers: " + p2A, (9 * canvasSizex) / 10, 110);

          p5.stroke("brown");
          p5.text("Wood: " + turn.players[0].wood, canvasSizex / 10, 140);
          p5.text("Wood: " + turn.players[1].wood, (9 * canvasSizex) / 10, 140);

          p5.stroke("grey");
          p5.text("Metal: " + turn.players[0].metal, canvasSizex / 10, 160);
          p5.text(
            "Metal: " + turn.players[1].metal,
            (9 * canvasSizex) / 10,
            160
          );
        }

        let update = false;
        let turn = 0;
        let ended = false;
        let pause = false;
        let tick = 0;
        let timer = 0;

        p5.draw = () => {
          p5.fill(0);
          p5.textSize(32);
          p5.textAlign(p5.CENTER);
          timer++;

          if (
            p5.mouseIsPressed &&
            timer > 10 &&
            p5.mouseX < canvasSizex / 2 + 50 &&
            p5.mouseX > canvasSizex / 2 - 50 &&
            p5.mouseY > (canvasSize * 12) / 16 + 10 &&
            p5.mouseY < (canvasSize * 12) / 16 + 50
          ) {
            timer = 0;
            pause = !pause;
            drawUi(turn + 1, game.turns.length, pause, false);
          }

          if (
            p5.mouseIsPressed &&
            timer > 10 &&
            p5.mouseX < canvasSizex / 2 - 100 &&
            p5.mouseX > canvasSizex / 2 - 200 &&
            p5.mouseY > (canvasSize * 12) / 16 + 10 &&
            p5.mouseY < (canvasSize * 12) / 16 + 50 &&
            turn > 0
          ) {
            timer = 0;
            turn--;
            drawUi(turn + 1, game.turns.length, pause, true);
            doPhase(game.turns[turn].attack, game.init.map.length);
            p5.fill(0);
            p5.textSize(32);
            p5.noStroke();
            p5.text(
              turn + 1 + "/" + game.turns.length,
              canvasSizex / 2,
              (canvasSize * 15) / 16
            );
            ended = false;
            tick = 0;
          }

          if (
            p5.mouseIsPressed &&
            timer > 10 &&
            p5.mouseX < canvasSizex / 2 + 200 &&
            p5.mouseX > canvasSizex / 2 + 100 &&
            p5.mouseY > (canvasSize * 12) / 16 + 10 &&
            p5.mouseY < (canvasSize * 12) / 16 + 50 &&
            turn + 1 < game.turns.length
          ) {
            timer = 0;
            turn++;
            drawUi(turn + 1, game.turns.length, pause, true);
            doPhase(game.turns[turn].attack, game.init.map.length);
            p5.fill(0);
            p5.textSize(32);
            p5.noStroke();
            p5.text(
              turn + 1 + "/" + game.turns.length,
              canvasSizex / 2,
              (canvasSize * 15) / 16
            );
            tick = 0;
          }

          if (!pause) {
            tick = p5.frameCount;
          } else {
            tick = 0;
          }

          if (!update) {
            p5.clear();
            p5.image(bg, (2 * canvasSizex) / 10, 0);
            console.log("updated");
            update = true;
          }

          if (!pause && !ended) {
            const poss = [];
            p5.image(bg, (2 * canvasSizex) / 10, 0);
            const mSize = game.init.map.length;
            for (let i = 0; i < game.turns[turn].move.units.length; i++) {
              const unit = game.turns[turn].move.units[i];
              let found = false;
              for (let j = 0; j < poss.length; j++) {
                if (
                  poss[j][0] == unit.position[0] &&
                  poss[j][1] == unit.position[1]
                ) {
                  found = true;
                }
              }
              if (!found) {
                poss.push(unit.position);
                const newx = game.turns[turn].collect.units[i].position[0];
                const newy = game.turns[turn].collect.units[i].position[1];
                const nx =
                  (newx * ((canvasSizex * 6) / 10)) / mSize +
                  (2 * canvasSizex) / 10 +
                  (canvasSizex * 6) / 10 / (2 * mSize);
                const ny =
                  (newy * ((canvasSize * 6) / 8)) / mSize +
                  (canvasSize * 6) / 8 / (2 * mSize);
                const ox =
                  (unit.position[0] * ((canvasSizex * 6) / 10)) / mSize +
                  (2 * canvasSizex) / 10 +
                  (canvasSizex * 6) / 10 / (2 * mSize);
                const oy =
                  (unit.position[1] * ((canvasSize * 6) / 8)) / mSize +
                  (canvasSize * 6) / 8 / (2 * mSize);
                let ttick = tick;
                if (tick % 30 == 0 || tick % 30 == 1) {
                  ttick = 29;
                }
                const x = ((ttick % 30) * (nx - ox)) / 30 + ox;
                const y = ((ttick % 30) * (ny - oy)) / 30 + oy;
                drawUnit(unit, x, y);
              }
            }
          }

          if (tick % 30 == 1 && !ended) {
            drawUi(turn + 1, game.turns.length, pause, true);
            p5.fill(0);
            p5.text(
              turn + 1 + "/" + game.turns.length,
              canvasSizex / 2,
              (canvasSize * 15) / 16
            );
            doPhase(game.turns[turn].attack, game.init.map.length);
            turn++;
            if (turn == game.turns.length) {
              ended = true;
              p5.noStroke();
              p5.fill(0);
              p5.text(
                "Game Over",
                canvasSizex / 2,
                40 + (canvasSize * 15) / 16
              );
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
