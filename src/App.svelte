<script>
  import { onMount } from "svelte";

  const boardSize = 9;
  let board = [...Array(boardSize)].map(() => Array(boardSize).fill(null));
  const pieces = ["🔴", "⚫️", "🔵", "🟣", "🟡", "🟤", "🟢", "🟠"];

  let nextPieces = [];
  let selectedSquareX;
  let selectedSquareY;

  function calculateNextPieces() {
    nextPieces = Array(3)
      .fill(null)
      .map(() => pieces[getRandomInt(pieces.length)]);
  }

  function placeNextPieces() {
    let numPlaced = 0;

    while (numPlaced < 3) {
      let x = getRandomInt(boardSize);
      let y = getRandomInt(boardSize);

      // if it's not taken by something
      if (board[x][y] == null) {
        // place a thing
        board[x][y] = nextPieces.pop();
        numPlaced++;
      }
    }
  }

  function getRandomInt(max) {
    return Math.floor(Math.random() * max);
  }

  onMount(() => {
    calculateNextPieces();
    placeNextPieces();
    calculateNextPieces();
  });

  function clicked(i, j) {
    // i is row, j is column
    console.log(i, j);
    console.log(board[i][j]);

    if (selectedSquareX == undefined && selectedSquareY == undefined) {
      // nothing is selected
      if (board[i][j]) {
        // something is there
        selectedSquareX = i;
        selectedSquareY = j;
      }
    }
    if (isBorderingSelectedSquare(i, j)) {
      // move the current selected dot there

      // just swap the two places
      let temp = board[i][j];
      board[i][j] = board[selectedSquareX][selectedSquareY];
      board[selectedSquareX][selectedSquareY] = temp;
      board = board;
      console.log(board);

      selectedSquareX = undefined;
      selectedSquareY = undefined;

      // make a new turn
      placeNextPieces();
      calculateNextPieces();
    } else {
      if (selectedSquareX && selectedSquareY) {
        console.log(selectedSquareX, selectedSquareY);
        if (board[i][j] != null) {
          console.log(board[selectedSquareX][selectedSquareY]);
          // just select it instead
          selectedSquareX = i;
          selectedSquareY = j;
          console.log(selectedSquareX, selectedSquareY);
        } else {
          selectedSquareX = undefined;
          selectedSquareY = undefined;
        }
      }
    }

    // if (board[i][j] == null) {
    //   selectedSquareX = undefined;
    //   selectedSquareY = undefined;
    //   return;
    // }

    // // if nothing was selected, select this square
    // if (selectedSquareX == undefined && selectedSquareY == undefined) {
    //   selectedSquareX = i;
    //   selectedSquareY = j;
    // }
    // else there is something at that square

    if (j + 1 < boardSize) {
      // ___
      // _0_
      // _*_
    }
    if (i + 1 < boardSize) {
    }
    if (j - 1 >= 0) {
    }
    if (i - 1 >= 0) {
    }

    // if (
    //   (!(j + 1 in board) || board[i][j + 1] == enemy) &&
    //   (!(j - 1 in board) || board[i][j - 1] == enemy) &&
    //   (!(i + 1 in board) || board[i + 1][j] == enemy) &&
    //   (!(i - 1 in board) || board[i - 1][j] == enemy)
    // ) {
    // }
  }

  function isBorderingSelectedSquare(i, j) {
    return (
      Math.abs(selectedSquareX - i) < 2 &&
      Math.abs(selectedSquareY - j) < 2 &&
      !(i == selectedSquareX && j == selectedSquareY)
    );
  }

  function hasFive(color) {
    var g = board;

    for (var i = 0; i < boardSize; i++) {
      for (var j = 0; j < boardSize; j++) {
        if (
          j + 4 in g &&
          g[i][j] == color &&
          g[i][j + 1] == color &&
          g[i][j + 2] == color &&
          g[i][j + 3] == color &&
          g[i][j + 4] == color
        )
          return true;
        else if (
          i + 4 in g &&
          g[i][j] == color &&
          g[i + 1][j] == color &&
          g[i + 2][j] == color &&
          g[i + 3][j] == color &&
          g[i + 4][j] == color
        )
          return true;
        else if (
          i + 4 in g &&
          j + 4 in g &&
          g[i][j] == color &&
          g[i + 1][j + 1] == color &&
          g[i + 2][j + 2] == color &&
          g[i + 3][j + 3] == color &&
          g[i + 4][j + 4] == color
        )
          return true;
        else if (
          i - 4 in g &&
          j + 4 in g &&
          g[i][j] == color &&
          g[i - 1][j + 1] == color &&
          g[i - 2][j + 2] == color &&
          g[i - 3][j + 3] == color &&
          g[i - 4][j + 4] == color
        )
          return true;
      }
    }
    return false;
  }

  import { quintOut } from "svelte/easing";
  import { crossfade } from "svelte/transition";

  const [send, receive] = crossfade({
    duration: (d) => Math.sqrt(d * 200),

    fallback(node, params) {
      const style = getComputedStyle(node);
      const transform = style.transform === "none" ? "" : style.transform;

      return {
        duration: 600,
        easing: quintOut,
        css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`,
      };
    },
  });

  // function checkWin() {
  //   for (let i = 0; i < boardSize; i++) {
  //     for (let j = 0; j < boardSize; j++) {
  // 			if
  // 		}
  //   }
  // }
</script>

<p>Next pieces: {nextPieces}</p>

<table>
  {#each board as row, i}
    <tr>
      {#each row as square, j}
        <td
          on:click={() => clicked(i, j)}
          class:selected={selectedSquareX === i && selectedSquareY === j}
          class:selected-border={Math.abs(selectedSquareX - i) < 2 &&
            Math.abs(selectedSquareY - j) < 2 &&
            !(i == selectedSquareX && j == selectedSquareY) &&
            board[i][j] == null}
          class:selected-border-occupied={Math.abs(selectedSquareX - i) < 2 &&
            Math.abs(selectedSquareY - j) < 2 &&
            !(i == selectedSquareX && j == selectedSquareY) &&
            board[i][j] != null}
        >
          {square ?? " "}
        </td>
      {/each}
    </tr>
  {/each}
</table>

<style>
  table tr td {
    width: 30px;
    height: 30px;
    border: 1px solid black;
  }
  td {
    text-align: center;
  }
  .selected {
    background-color: #fff23a;
  }
  .selected-border {
    background-color: #ffa689;
  }
  .selected-border-occupied {
    background-color: #ffac3e;
  }
</style>
