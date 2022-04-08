<script>
  import { onMount } from "svelte";

  const boardSize = 9;
  let board = [...Array(boardSize)].map(() => Array(boardSize).fill(null));
  const pieces = ["🔴", "⚫️", "🔵", "🟣", "🟡", "🟤", "🟢", "🟠"];

  let nextPieces = [];
  let selectedSquareX;
  let selectedSquareY;

  let reachableCells = {};

  let score = 0;
  let gameFinished = false;

  // whenever user clicks on a marble, we get all cells in reach to
  function allReachableCells(startPos) {
    // copy 2d array
    let grid = board.map(function (arr) {
      return arr.slice();
    });

    const queue = [{ coord: startPos, dist: 1 }];
    let reachableCells = {};
    const directs = [
      [-1, 0],
      [0, 1],
      [1, 0],
      [0, -1],
    ];
    const N = grid.length;
    const isValidCoord = (x, y) => x >= 0 && x < N && y >= 0 && y < N;

    // change to any
    grid[startPos[0]][startPos[1]] = 1;

    while (queue.length) {
      const {
        coord: [x, y],
        dist,
      } = queue.shift();

      reachableCells[`${x},${y}`] = true;

      for (let [moveX, moveY] of directs) {
        const nextX = x + moveX;
        const nextY = y + moveY;

        if (isValidCoord(nextX, nextY) && grid[nextX][nextY] == null) {
          queue.push({ coord: [nextX, nextY], dist: dist + 1 });
          grid[nextX][nextY] = 1;
        }
      }
    }

    // remove the starting cell from this dict
    delete reachableCells[`${startPos[0]},${startPos[1]}`];

    return reachableCells;
  }

  function shortestPathBinaryMatrix(grid, startPos, endPos) {
    if (grid[startPos[0]][startPos[1]]) return -1;

    const queue = [{ coord: startPos, dist: 1 }];
    const directs = [
      [-1, -1],
      [-1, 0],
      [-1, 1],
      [0, 1],
      [1, 1],
      [1, 0],
      [1, -1],
      [0, -1],
    ];
    const N = grid.length;
    const isValidCoord = (x, y) => x >= 0 && x < N && y >= 0 && y < N;

    // change to any
    grid[startPos[0]][startPos[1]] = 1;

    while (queue.length) {
      const {
        coord: [x, y],
        dist,
      } = queue.shift();

      // change to any destination
      if (x === endPos[0] && y === endPos[1]) {
        return dist;
      }

      for (let [moveX, moveY] of directs) {
        const nextX = x + moveX;
        const nextY = y + moveY;

        if (isValidCoord(nextX, nextY) && grid[nextX][nextY] === 0) {
          queue.push({ coord: [nextX, nextY], dist: dist + 1 });
          grid[nextX][nextY] = 1;
        }
      }
    }

    return -1;
  }

  function calculateNextPieces() {
    nextPieces = Array(3)
      .fill(null)
      .map(() => pieces[getRandomInt(pieces.length)]);
  }
  function placeNextPieces() {
    let numPlaced = 0;

    while (numPlaced < 3) {
      // if there is no more room (aka you placed 2 stones and there is one more to place)
      // aka there are no more nulls
      // just end the game
      let boardIsFull = true;

      for (let i = 0; i < boardSize; i++) {
        for (let j = 0; j < boardSize; j++) {
          if (board[i][j] == null) {
            boardIsFull = false;
          }
        }
      }

      if (boardIsFull) {
        gameFinished = true;
        return;
      }

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
    newTurn();
  });

  function newTurn() {
    // check if there are five in a row and delete them and increment score if so
    for (const piece of pieces) {
      let ret = hasFive(piece);
      // console.log(board);
      // console.log(ret);
      if (ret) {
        // returned 5 cells with a row of 5
        // just delete those cells then
        for (const cell of ret) {
          board[cell[0]][cell[1]] = null;
        }

        score += 5 + (ret.length - 5) * 5;

        // if you managed to get 5 in a row, skip putting more marbles down and just
        // let the user play again
        return;
      }
    }

    // make a new turn
    placeNextPieces();
    calculateNextPieces();

    // check if any of the new pieces that got added created more 5 in a rows
    for (const piece of pieces) {
      let ret = hasFive(piece);
      // console.log(ret);
      if (ret) {
        // returned 5 cells with a row of 5
        // just delete those cells then
        for (const cell of ret) {
          board[cell[0]][cell[1]] = null;
        }

        score += 5 + (ret.length - 5) * 5;
      }
    }

    gameFinished = true;
    // check if the entire board is filled with dots
    // then the game is over and you can display the score
    for (var i = 0; i < boardSize; i++) {
      for (var j = 0; j < boardSize; j++) {
        if (board[i][j] == null) {
          gameFinished = false;
        }
      }
    }
  }

  function selectedACell(i, j) {
    // reset reachable cells
    reachableCells = {};

    selectedSquareX = i;
    selectedSquareY = j;

    // selecting a cell will also calculate all of the cells that can be reached from that cell
    reachableCells = allReachableCells([i, j]);
    console.log(reachableCells);
    console.log(board);
  }

  function deselectCell() {
    selectedSquareX = undefined;
    selectedSquareY = undefined;
    // reset reachable cells
    reachableCells = {};
  }

  // i is row, j is column
  function clicked(i, j) {
    // nothing had been selected before (only happens once in the beginning of each turn)
    if (selectedSquareX == undefined && selectedSquareY == undefined) {
      // something is where we clicked
      if (board[i][j]) {
        // select that thing instead of what we had selected before

        selectedACell(i, j);
      }
    }
    // something had been selected before
    else {
      // something is where we clicked
      if (board[i][j]) {
        if (i == selectedSquareX && j == selectedSquareY) {
          // you clicked on your own selected marble again, so just deselect it (toggle)
          deselectCell();
        } else {
          // just select that thing instead of what we had selected before
          selectedACell(i, j);
        }
      }
      // there is nothing where we clicked
      else {
        // make sure that the cell that we clicked is a reachable cell from the current cell
        if (`${i},${j}` in reachableCells) {
          // move the previously selected cell to that cell!
          board[i][j] = board[selectedSquareX][selectedSquareY];
          board[selectedSquareX][selectedSquareY] = null;
          // board = board;
          selectedSquareX = undefined;
          selectedSquareY = undefined;

          // reset reachable cells
          reachableCells = {};

          newTurn();
        } else {
          // if you clicked somewhere where you can't move, just deselect
          deselectCell();
        }
      }
    }
  }

  // better hasFive would start at a point and move
  function betterHasFive(color) {
    var g = board;

    const directions = [
      [1, 0],
      [0, 1],
      [1, 1],
      [-1, 1],
    ];
    const isValidCoord = (x, y) =>
      x >= 0 && x < boardSize && y >= 0 && y < boardSize;

    for (var i = 0; i < boardSize; i++) {
      for (var j = 0; j < boardSize; j++) {
        for (const [moveX, moveY] of directions) {
          // for each cell, for each direction, add the direction to it until it stops
          // being equal to the color, and then if its 5 or greater, return the cells
          // else return false

          let nextX = i + moveX;
          let nextY = j + moveY;

          // if (isValidCoord(nextX, nextY) && board[nextX][nextY] == color){
          //   queueMicrotask.push({coord: [nextX, nextY], dist: dist+1})

          // }
          let dist = 0;
          // maybe better as a while loop because then you can say
          while (isValidCoord(nextX, nextY) && g[nextX][nextY] == color) {
            nextX = i + moveX;
            nextY = j + moveY;
            dist++;
          }

          // after the loop, you would have found the longest chain of the same marbles
          // in the same direction, thus you can use the dist to add points
          if (dist > 4) {
            // you might want to have the array of cells in here to return tho...
            return Array(dist)
              .fill([i, j])
              .map((e, i) => e.map((e, i2) => e + i * [moveX, moveY][i2]));
          }
        }
      }
    }

    return false;
  }

  function hasFive2(color) {
    var g = board;

    const isValidCoord = (x, y) =>
      x >= 0 && x < boardSize && y >= 0 && y < boardSize;

    const directions = [
      [1, 0],
      [0, 1],
      [1, 1],
      [-1, 1],
    ];

    for (var i = 0; i < boardSize; i++) {
      for (var j = 0; j < boardSize; j++) {
        // check if the current index is the color in the first place
        if (g[i][j] == color) {
          // then if so, check for strings in all directions from this point

          for (const direction of directions) {
            let dist = 0;
            let nextCell = [i, j];

            // debugger;

            // next cell is the new cell to check
            while (
              isValidCoord(nextCell[0], nextCell[1]) &&
              g[nextCell[0]][nextCell[1]] == color
            ) {
              nextCell = [i + direction[0], j + direction[1]];
              dist++;
            }

            // we've reached the maximum length of the string
            // if it's larger than 4 in distance, return the length (for now)
            // else return 0
            if (dist > 4) {
              return dist;
            }
          }
        }
      }
    }
    return false;
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
          if (j + 5 in g && g[i][j + 5] == color)
            if (j + 6 in g && g[i][j + 6] == color)
              if (j + 7 in g && g[i][j + 7] == color)
                if (j + 8 in g && g[i][j + 8] == color)
                  return [
                    [i, j],
                    [i, j + 1],
                    [i, j + 2],
                    [i, j + 3],
                    [i, j + 4],
                    [i, j + 5],
                    [i, j + 6],
                    [i, j + 7],
                    [i, j + 8],
                  ];
                else
                  return [
                    [i, j],
                    [i, j + 1],
                    [i, j + 2],
                    [i, j + 3],
                    [i, j + 4],
                    [i, j + 5],
                    [i, j + 6],
                    [i, j + 7],
                  ];
              else
                return [
                  [i, j],
                  [i, j + 1],
                  [i, j + 2],
                  [i, j + 3],
                  [i, j + 4],
                  [i, j + 5],
                  [i, j + 6],
                ];
            else
              return [
                [i, j],
                [i, j + 1],
                [i, j + 2],
                [i, j + 3],
                [i, j + 4],
                [i, j + 5],
              ];
          else
            return [
              [i, j],
              [i, j + 1],
              [i, j + 2],
              [i, j + 3],
              [i, j + 4],
            ];
        else if (
          i + 4 in g &&
          g[i][j] == color &&
          g[i + 1][j] == color &&
          g[i + 2][j] == color &&
          g[i + 3][j] == color &&
          g[i + 4][j] == color
        )
          if (i + 5 in g && g[i + 5][j] == color)
            if (i + 6 in g && g[i + 6][j] == color)
              if (i + 7 in g && g[i + 7][j] == color)
                if (i + 8 in g && g[i + 8][j] == color)
                  return [
                    [i, j],
                    [i + 1, j],
                    [i + 2, j],
                    [i + 3, j],
                    [i + 4, j],
                    [i + 5, j],
                    [i + 6, j],
                    [i + 7, j],
                    [i + 8, j],
                  ];
                else
                  return [
                    [i, j],
                    [i + 1, j],
                    [i + 2, j],
                    [i + 3, j],
                    [i + 4, j],
                    [i + 5, j],
                    [i + 6, j],
                    [i + 7, j],
                  ];
              else
                return [
                  [i, j],
                  [i + 1, j],
                  [i + 2, j],
                  [i + 3, j],
                  [i + 4, j],
                  [i + 5, j],
                  [i + 6, j],
                ];
            else
              return [
                [i, j],
                [i + 1, j],
                [i + 2, j],
                [i + 3, j],
                [i + 4, j],
                [i + 5, j],
              ];
          else
            return [
              [i, j],
              [i + 1, j],
              [i + 2, j],
              [i + 3, j],
              [i + 4, j],
            ];
        else if (
          i + 4 in g &&
          j + 4 in g &&
          g[i][j] == color &&
          g[i + 1][j + 1] == color &&
          g[i + 2][j + 2] == color &&
          g[i + 3][j + 3] == color &&
          g[i + 4][j + 4] == color
        )
          if (i + 5 in g && j + 5 in g && g[i + 5][j + 5] == color)
            if (i + 6 in g && j + 6 in g && g[i + 6][j + 6] == color)
              if (i + 7 in g && j + 7 in g && g[i + 7][j + 7] == color)
                if (i + 8 in g && j + 8 in g && g[i + 8][j + 8] == color)
                  return [
                    [i, j],
                    [i + 1, j + 1],
                    [i + 2, j + 2],
                    [i + 3, j + 3],
                    [i + 4, j + 4],
                    [i + 5, j + 5],
                    [i + 6, j + 6],
                    [i + 7, j + 7],
                    [i + 8, j + 8],
                  ];
                else
                  return [
                    [i, j],
                    [i + 1, j + 1],
                    [i + 2, j + 2],
                    [i + 3, j + 3],
                    [i + 4, j + 4],
                    [i + 5, j + 5],
                    [i + 6, j + 6],
                    [i + 7, j + 7],
                  ];
              else
                return [
                  [i, j],
                  [i + 1, j + 1],
                  [i + 2, j + 2],
                  [i + 3, j + 3],
                  [i + 4, j + 4],
                  [i + 5, j + 5],
                  [i + 6, j + 6],
                ];
            else
              return [
                [i, j],
                [i + 1, j + 1],
                [i + 2, j + 2],
                [i + 3, j + 3],
                [i + 4, j + 4],
                [i + 5, j + 5],
              ];
          else
            return [
              [i, j],
              [i + 1, j + 1],
              [i + 2, j + 2],
              [i + 3, j + 3],
              [i + 4, j + 4],
            ];
        else if (
          i - 4 in g &&
          j + 4 in g &&
          g[i][j] == color &&
          g[i - 1][j + 1] == color &&
          g[i - 2][j + 2] == color &&
          g[i - 3][j + 3] == color &&
          g[i - 4][j + 4] == color
        )
          if (i - 5 in g && j + 5 in g && g[i - 5][j + 5] == color)
            if (i - 6 in g && j + 6 in g && g[i - 6][j + 6] == color)
              if (i - 7 in g && j + 7 in g && g[i - 7][j + 7] == color)
                if (i - 8 in g && j + 8 in g && g[i - 8][j + 8] == color)
                  return [
                    [i, j],
                    [i - 1, j + 1],
                    [i - 2, j + 2],
                    [i - 3, j + 3],
                    [i - 4, j + 4],
                    [i - 5, j + 5],
                    [i - 6, j + 6],
                    [i - 7, j + 7],
                    [i - 8, j + 8],
                  ];
                else
                  return [
                    [i, j],
                    [i - 1, j + 1],
                    [i - 2, j + 2],
                    [i - 3, j + 3],
                    [i - 4, j + 4],
                    [i - 5, j + 5],
                    [i - 6, j + 6],
                    [i - 7, j + 7],
                  ];
              else
                return [
                  [i, j],
                  [i - 1, j + 1],
                  [i - 2, j + 2],
                  [i - 3, j + 3],
                  [i - 4, j + 4],
                  [i - 5, j + 5],
                  [i - 6, j + 6],
                ];
            else
              return [
                [i, j],
                [i - 1, j + 1],
                [i - 2, j + 2],
                [i - 3, j + 3],
                [i - 4, j + 4],
                [i - 5, j + 5],
              ];
          else
            return [
              [i, j],
              [i - 1, j + 1],
              [i - 2, j + 2],
              [i - 3, j + 3],
              [i - 4, j + 4],
            ];
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
</script>

<main>
  <h1>Marbles</h1>
  <div id="board">
    {#each board as row, i}
      {#each row as square, j}
        <div
          id="cell"
          on:click={() => clicked(i, j)}
          class:selected={selectedSquareX === i && selectedSquareY === j}
          class:reachable={reachableCells[`${i},${j}`]}
        >
          <div class="content">
            {square ?? " "}
          </div>
        </div>
      {/each}
    {/each}
  </div>

  {#if gameFinished}
    <h2>Game Over. Score: {score}</h2>
  {:else}
    <h3>Next pieces: {nextPieces}</h3>
    <h3>Score: {score}</h3>
  {/if}
</main>

<style>
  main {
    max-width: 1000px;
    margin: 0 auto;
    text-align: center;
  }

  #board {
    max-width: 700px;
    display: grid;
    margin: 0 auto;
    grid-template-columns: repeat(9, 1fr);
    grid-template-rows: repeat(9, 1fr);
    border: 1px solid black;
  }

  #cell {
    border: 1px solid black;
    position: relative;
  }
  #cell:after {
    content: "";
    display: block;
    padding-bottom: 100%;
  }
  .content {
    position: absolute;
    width: 100%;
    height: 100%;

    display: flex;
    align-items: center;
    justify-content: center;
    font-size: min(7vmin, 50px);
  }
  .selected {
    background-color: #fff23a;
  }

  .reachable {
    background-color: #fffbc1;
  }
</style>
