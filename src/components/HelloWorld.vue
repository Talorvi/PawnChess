<template>
  <div>
    <div class="hello">
      <div class="board" id="board1"></div>
    </div>
  </div>
</template>

<script>
import {Chessboard} from "cm-chessboard";
import {BORDER_TYPE} from "cm-chessboard/src/cm-chessboard/Chessboard";

export default {
  name: 'HelloWorld',
  mounted() {
    new Chessboard(document.getElementById("board1"),
      {
        position: "rn2k1r1/ppp1pp1p/3p2p1/5bn1/P7/2N2B2/1PPPPP2/2BNK1RR",
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
  }
}
</script>
<style>
  .board {
    height: 800px;
    width: 800px;
  }
</style>