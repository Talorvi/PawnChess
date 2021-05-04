<template>
  <div>
    <div class="hello">
      <div class="board" id="board1"></div>
      <div class="leaderboard" id="leaderboard">
        <div v-bind:class="{ active: !turn }">White: {{points.black}}</div>
        <div v-bind:class="{ active: turn }">Black: {{points.white}}</div>
      </div>
      <div id="buttons">
        <button type="button" v-on:click="turnAround">Turn around</button>
      </div>
    </div>
  </div>
</template>

<script>
import {Chessboard} from "cm-chessboard";
import {BORDER_TYPE, COLOR, INPUT_EVENT_TYPE} from "cm-chessboard/src/cm-chessboard/Chessboard";

export default {
  name: 'HelloWorld',
  data() {
    return {
      chessboard: null,
      points: {
        white: 0,
        black: 0
      },
      turn: 0
    }
  },
  methods: {
    initBoard: function() {
      this.chessboard = new Chessboard(document.getElementById("board1"),
              {
                position: "pppppppp/pppppppp/8/8/8/8/PPPPPPPP/PPPPPPPP",
                sprite: {
                  url: require("@/assets/images/chessboard-sprite-staunty.svg")
                },
                style: {
                  cssClass: "default",
                  showCoordinates: true, // show ranks and files
                  borderType: BORDER_TYPE.thin, // thin: thin border, frame: wide border with coordinates in it, none: no border
                  aspectRatio: 1 // height/width. Set to `undefined`, if you want to define it only in the css.
                }
              });
      this.whiteMove();
      //console.log(this.chessboard);
    },
    whiteMove: function () {
      this.chessboard.disableMoveInput();
      this.chessboard.enableMoveInput((event) => {
        switch (event.type) {
          case INPUT_EVENT_TYPE.moveStart:
            //console.log(`moveStart: ${event.square}`);
            // return `true`, if input is accepted/valid, `false` aborts the interaction, the piece will not move
            return true;
          case INPUT_EVENT_TYPE.moveDone:
            //console.log(`moveDone: ${event.squareFrom}-${event.squareTo}`);

            // eslint-disable-next-line no-case-declarations
            let result = this.validateMove(event.squareFrom, event.squareTo, 0);

            if (result) {
              this.checkIfEdge(event.squareTo, 0);
              this.blackMove();
              this.turn = !this.turn;
              return result;
            }

            return false;

          case INPUT_EVENT_TYPE.moveCanceled:
            //console.log(`moveCanceled`)
        }
      }, COLOR.white)
    },
    blackMove: function () {
      this.chessboard.disableMoveInput();
      this.chessboard.enableMoveInput((event) => {
        switch (event.type) {
          case INPUT_EVENT_TYPE.moveStart:
            //console.log(`moveStart: ${event.square}`);
            // return `true`, if input is accepted/valid, `false` aborts the interaction, the piece will not move
            return true;
          case INPUT_EVENT_TYPE.moveDone:
            //console.log(`moveDone: ${event.squareFrom}-${event.squareTo}`);

            // eslint-disable-next-line no-case-declarations
            let result = this.validateMove(event.squareFrom, event.squareTo, 1);

            if (result) {
              this.checkIfEdge(event.squareTo, 1);
              this.whiteMove();
              this.turn = !this.turn;
              return result;
            }

            return false;
          case INPUT_EVENT_TYPE.moveCanceled:
            //console.log(`moveCanceled`)
        }
      }, COLOR.black)
    },
    // eslint-disable-next-line no-unused-vars
    validateMove(squareFrom, squareTo, player) {
      return true;
    },
    addPoint(player) {
      if (!player) {
        this.points.black++;
      }
      else {
        this.points.white++;
      }
      console.log(this.points);
    },
    checkIfEdge(square, player) {
      if (square.includes("1") || square.includes("8")) {
          this.addPoint(player);
      }
    },
    turnAround: function () {
      this.chessboard.setOrientation(this.chessboard.getOrientation() === 'w' ? 'b' : 'w');
    }
  },
  mounted() {
    this.initBoard();
  }
}
</script>
<style>
  .board {
    max-width: 500px;
    max-height: 100%;
  }
  .active {
    font-weight: bold;
  }
</style>