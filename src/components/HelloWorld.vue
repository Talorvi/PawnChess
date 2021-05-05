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
      turn: 0,
      notMoved: new Set([
        'a1','b1','c1','d1','e1','f1','g1','h1',
        'a2','b2','c2','d2','e2','f2','g2','h2',
        'a7','b7','c7','d7','e7','f7','g7','h7',
        'a8','b8','c8','d8','e8','f8','g8','h8']),
      board: {
        height: ['1', '2', '3', '4', '5', '6', '7', '8'],
        width: ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
      },
      ep: undefined,
      epturn: 0
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
    },
    whiteMove: function (squareTo = null) {
      if (squareTo) {
        this.chessboard.setPiece(squareTo, undefined);
      }

      let destroyedPawn = null;
      this.chessboard.disableMoveInput();
      this.chessboard.enableMoveInput((event) => {
        switch (event.type) {
          case INPUT_EVENT_TYPE.moveStart:
            this.checkIfEnd();
            //console.log(`moveStart: ${event.square}`);
            // return `true`, if input is accepted/valid, `false` aborts the interaction, the piece will not move
            return true;
          case INPUT_EVENT_TYPE.moveDone:
            //console.log(`moveDone: ${event.squareFrom}-${event.squareTo}`);

            // eslint-disable-next-line no-case-declarations
            let result = this.validateMove(event.squareFrom, event.squareTo, 0);
            this.epturn++;

            if (result) {
              let edge = this.checkIfEdge(event.squareTo, 0);

              if (edge) {
                destroyedPawn = event.squareTo;
              }
              this.blackMove(destroyedPawn);
              this.turn = !this.turn;
              return result;
            }

            return false;

          case INPUT_EVENT_TYPE.moveCanceled:
            //console.log(`moveCanceled`)
        }
      }, COLOR.white);
    },
    blackMove: function (squareTo = null) {
      if (squareTo) {
        this.chessboard.setPiece(squareTo, undefined);
      }

      let destroyedPawn = null;
      this.chessboard.disableMoveInput();
      this.chessboard.enableMoveInput((event) => {
        switch (event.type) {
          case INPUT_EVENT_TYPE.moveStart:
            this.checkIfEnd();
            //console.log(`moveStart: ${event.square}`);
            // return `true`, if input is accepted/valid, `false` aborts the interaction, the piece will not move
            return true;
          case INPUT_EVENT_TYPE.moveDone:
            //console.log(`moveDone: ${event.squareFrom}-${event.squareTo}`);

            // eslint-disable-next-line no-case-declarations
            let result = this.validateMove(event.squareFrom, event.squareTo, 1);
            this.epturn++;

            if (result) {
              let edge = this.checkIfEdge(event.squareTo, 1);

              if (edge) {
                destroyedPawn = event.squareTo;
              }

              this.whiteMove(destroyedPawn);
              this.turn = !this.turn;
              return result;
            }

            return false;
          case INPUT_EVENT_TYPE.moveCanceled:
            //console.log(`moveCanceled`)
        }
      }, COLOR.black);
    },
    validateMove(squareFrom, squareTo, player, check=false) {
      let colF = squareFrom.slice(0,1).charCodeAt(0);
      let rowF = squareFrom.slice(-1);
      let colT = squareTo.slice(0,1).charCodeAt(0);
      let rowT = squareTo.slice(-1);
      if(player){
        if(rowT<rowF){
          if(colT===colF){
            if((rowF-rowT===1 || (rowF-rowT===2 && this.notMoved.has(squareFrom)))
                    && this.chessboard.getPiece(squareTo)===undefined){
              if(rowF-rowT===2) {this.ep = squareTo;this.epturn=0;}
              if(!check)
                this.notMoved.delete(squareFrom);
              return true;
            }}
          else if(Math.abs(colT-colF)===1 && rowF-rowT===1){
            if(this.chessboard.getPiece(squareTo)==='wp'){
              if(!check){
                this.notMoved.delete(squareFrom);
                this.notMoved.delete(squareTo);
              }
              return true;
            }
            else if(this.ep===squareTo[0]+(parseInt(rowT)+1) && this.epturn===1){
              this.chessboard.setPiece(this.ep,undefined);
              return true;
            }
          }
        }
      }
      else{
        if(rowT>rowF){
          if(colT===colF){
            if((rowT-rowF===1 || (rowT-rowF===2 && this.notMoved.has(squareFrom)))
                    && this.chessboard.getPiece(squareTo)===undefined){
              if(rowT-rowF===2) {this.ep = squareTo;this.epturn=0;}
              if(!check)
                this.notMoved.delete(squareFrom);
              return true;
            }}
          else if(Math.abs(colT-colF)===1 && rowT-rowF===1){
            if(this.chessboard.getPiece(squareTo)==='bp'){
              if(!check){
                this.notMoved.delete(squareFrom);
                this.notMoved.delete(squareTo);
              }
              return true;
            }
            else if(this.ep===squareTo[0]+(parseInt(rowT)-1) && this.epturn===1){
              this.chessboard.setPiece(this.ep,undefined);
              return true;
            }
          }
        }
      }
      return false;
    },
    addPoint(player) {
      if (!player) {
        this.points.black++;
      }
      else {
        this.points.white++;
      }
    },
    checkIfEdge(square, player) {
      if (square.includes("1") || square.includes("8")) {
          this.addPoint(player);
          return true;
      }
      return false;
    },
    turnAround: function () {
      this.chessboard.setOrientation(this.chessboard.getOrientation() === 'w' ? 'b' : 'w');
    },
    checkIfEnd: function () {
      let whiteValidMoves = 0;
      let blackValidMoves = 0;
      let gameFinished = false;
      this.board.width.forEach((w, i) => {
        this.board.height.forEach((h, j) => {
          let pawn = this.chessboard.getPiece(w+h);
          if (pawn) {
            if (pawn === "wp") {
              let moves;
              if (this.notMoved.has(w+h)) {
                moves = [
                  w+this.board.height[j+1], this.board.width[i-1]+this.board.height[j+1], this.board.width[i+1]+this.board.height[j+1],
                  w+this.board.height[j+2]
                ];
              } else {
                moves = [
                  w+this.board.height[j+1], this.board.width[i-1]+this.board.height[j+1], this.board.width[i+1]+this.board.height[j+1]
                ];
              }
              moves.forEach(move => {
                try {
                  let properMove = this.validateMove(w+h, move, 0, true);
                  if (properMove) {
                    whiteValidMoves++;
                  }
                } catch (e) {
                  console.log("Unexpected error");
                }
              });
            } else if (pawn === "bp") {
              let moves;
              if (this.notMoved.has(w+h)) {
                moves = [
                  w+this.board.height[j-1], this.board.width[i-1]+this.board.height[j-1], this.board.width[i+1]+this.board.height[j-1],
                  w+this.board.height[j-2]
                ];
              } else {
                moves = [
                  w+this.board.height[j-1], this.board.width[i-1]+this.board.height[j-1], this.board.width[i+1]+this.board.height[j-1]
                ];
              }
              moves.forEach(move => {
                try {
                  let properMove = this.validateMove(w+h, move, 1, true);
                  if (properMove) {
                    blackValidMoves++;
                  }
                } catch (e) {
                  console.log("Unexpected error");
                }
              });
            }
          }
        });
      });
      console.log("White valid moves: " + whiteValidMoves);
      console.log("Black valid moves: " + blackValidMoves);
      if (!this.turn && !whiteValidMoves) {
        console.log("Koniec gry od białych");
        gameFinished = true;
      }
      else if (this.turn && !blackValidMoves) {
        console.log("Koniec gry od czarnych");
        gameFinished = true;
      }
      if (gameFinished) {
        this.chessboard.disableMoveInput();
        if (this.points.white > this.points.black) {
          alert("Białe wygrały!");
        }
        else if (this.points.white < this.points.black) {
          alert("Czarne wygrały!");
        }
        else {
          alert("Remis!");
        }
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
    max-width: 500px;
    max-height: 100%;
  }
  .active {
    font-weight: bold;
  }
</style>