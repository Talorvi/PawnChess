<template>
  <div>
    <div class="hello">
      <div class="board" id="board1"></div>
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
      }
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
      console.log(this.chessboard);
    },
    whiteMove: function () {
      this.chessboard.disableMoveInput();
      this.chessboard.enableMoveInput((event) => {
        switch (event.type) {
          case INPUT_EVENT_TYPE.moveStart:
            console.log(`moveStart: ${event.square}`);
            // return `true`, if input is accepted/valid, `false` aborts the interaction, the piece will not move
            return true;
          case INPUT_EVENT_TYPE.moveDone:
            console.log(`moveDone: ${event.squareFrom}-${event.squareTo}`);

            // eslint-disable-next-line no-case-declarations
            let result = this.validateMove(event.squareFrom, event.squareTo, 0);

            if (result) {
              this.checkIfEdge(event.squareTo, 0);
              this.blackMove();
              return result;
            }

            return false;

          case INPUT_EVENT_TYPE.moveCanceled:
            console.log(`moveCanceled`)
        }
      }, COLOR.white)
    },
    blackMove: function () {
      this.chessboard.disableMoveInput();
      this.chessboard.enableMoveInput((event) => {
        switch (event.type) {
          case INPUT_EVENT_TYPE.moveStart:
            console.log(`moveStart: ${event.square}`);
            // return `true`, if input is accepted/valid, `false` aborts the interaction, the piece will not move
            return true;
          case INPUT_EVENT_TYPE.moveDone:
            console.log(`moveDone: ${event.squareFrom}-${event.squareTo}`);

            // eslint-disable-next-line no-case-declarations
            let result = this.validateMove(event.squareFrom, event.squareTo, 1);

            if (result) {
              this.checkIfEdge(event.squareTo, 1);
              this.whiteMove();
              return result;
            }

            return false;
          case INPUT_EVENT_TYPE.moveCanceled:
            console.log(`moveCanceled`)
        }
      }, COLOR.black)
    },
    // eslint-disable-next-line no-unused-vars
    validateMove(squareFrom, squareTo, player) {
      return true;
    },
    addPoint(player) {
      if (!player) {
        this.points.white++;
      }
      else {
        this.points.black++;
      }
      console.log(this.points);
    },
    checkIfEdge(square, player) {
      if (square.includes("1") || square.includes("8")) {
          this.addPoint(player);
      }
    }
  },
  mounted() {
    this.initBoard();
  }
}
</script>
<style>
  .board {
    height: 500px;
    width: 500px;
  }
</style>