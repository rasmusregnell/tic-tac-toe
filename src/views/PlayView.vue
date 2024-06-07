<script setup lang="ts">
import { ref } from "vue";

interface PingResponse {
  pong: string;
}

const player = ref([
  [false, false, false],
  [false, false, false],
  [false, false, false],
]);

const computer = ref([
  [false, false, false],
  [false, false, false],
  [false, false, false],
]);

const clickable = ref(true);
const has_won = ref(false);
const has_lost = ref(false);
const tie = ref(false);

// transform board to a flattened array, which fits format in python
function transform_board(): number[] {
  const board: number[][] = Array.from({ length: 3 }, () => Array(3).fill(0));
  for (let y = 0; y < 3; y++) {
    for (let x = 0; x < 3; x++) {
      if (player.value[y][x]) {
        board[y][x] = 1;
      }
      if (computer.value[y][x]) {
        board[y][x] = 2;
      }
    }
  }
  return board.flat(1);
}

// updates player and computer arrays based on array in flattened format
function update_board(arr: number[]): void {
  const board: number[][] = [];
  // turn into 2d array
  for (let i = 0; i < arr.length; i += 3) {
    board.push(arr.slice(i, i + 3));
  }

  // update player and computer
  for (let y = 0; y < 3; y++) {
    for (let x = 0; x < 3; x++) {
      if (board[y][x] == 1) {
        player.value[y][x] = true;
      }
      if (board[y][x] == 2) {
        computer.value[y][x] = true;
      }
    }
  }
}

// model from rest makes move, updates board
async function model_move() {
  // board is transfromed to fit expected format number[]
  const board = transform_board();

  try {
    //the board is sent to REST
    const response = await fetch("http://localhost:5175/sendBoard", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(board),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }

    // New board in flattened format recieved from REST
    const responseData = await response.json(); // Parse response JSON

    //console.log("Data sent successfully!");
    //console.log("Received response:", responseData);

    // we update the board
    update_board(responseData);
  } catch (error) {
    console.error("Error sending data and receiving response:", error);
    throw error; // Propagate error to the caller
  }
}

function check_has_won(board: boolean[][]): boolean {
  // Check rows
  for (let i = 0; i < 3; i++) {
    if (board[i][0] && board[i][1] && board[i][2]) {
      return true;
    }
  }

  // Check columns
  for (let j = 0; j < 3; j++) {
    if (board[0][j] && board[1][j] && board[2][j]) {
      return true;
    }
  }

  // Check diagonals
  if (board[0][0] && board[1][1] && board[2][2]) {
    return true;
  }
  if (board[0][2] && board[1][1] && board[2][0]) {
    return true;
  }

  // If no three consecutive trues are found
  return false;
}

function get_possible_moves(): Array<Array<number>> {
  const possible_moves = [];
  for (let y = 0; y < 3; y++) {
    for (let x = 0; x < 3; x++) {
      if (!player.value[y][x] && !computer.value[y][x])
        possible_moves.push([y, x]);
    }
  }
  return possible_moves;
}

function random_computer_move() {
  const possible_moves = get_possible_moves();
  const index =
    possible_moves[Math.floor(Math.random() * possible_moves.length)];
  computer.value[index[0]][index[1]] = true;
}

function arraysEqual(a1: number[], a2: number[]): boolean {
  if (a1.length !== a2.length) {
    return false;
  }
  for (let i = 0; i < a1.length; i++) {
    if (a1[i] !== a2[i]) {
      return false;
    }
  }
  return true;
}

//returns true if game is done, updates values on game state
function check_condition(actor: Array<Array<boolean>>, player: boolean) {
  if (check_has_won(actor) && player) {
    has_won.value = true;
    return true;
  }
  if (check_has_won(actor) && !player) {
    has_lost.value = true;
    return true;
  }
  if (get_possible_moves().length == 0) {
    tie.value = true;
    return true;
  }
  return false;
}

function round(x: number, y: number) {
  // check if there exists possible moves
  if (!get_possible_moves().find((e) => arraysEqual(e, [y - 1, x - 1]))) {
    console.log("no possible moves found");
    return;
  }

  player.value[y - 1][x - 1] = true;
  if (check_condition(player.value, true)) {
    return;
  }

  clickable.value = false;
  setTimeout(() => {
    model_move().then((e) => check_condition(computer.value, false));
    clickable.value = true;
  }, 700);
}

function restart() {
  computer.value = [
    [false, false, false],
    [false, false, false],
    [false, false, false],
  ];

  player.value = [
    [false, false, false],
    [false, false, false],
    [false, false, false],
  ];

  has_won.value = false;
  has_lost.value = false;
  tie.value = false;
}
</script>

<template>
  <div class="w-full h-full flex flex-col justify-center items-center">
    <div class="w-full flex justify-center">
      <span v-if="has_won" class="mb-8 text-dark_green text-2xl">You Won!</span>
    </div>
    <div class="w-full flex justify-center">
      <span v-if="tie" class="mb-8 text-dark_green text-2xl">It is a tie!</span>
    </div>
    <div class="w-full flex justify-center">
      <span v-if="has_lost" class="mb-8 text-[#ff0000] text-2xl"
        >You Lost!</span
      >
    </div>
    <div class="w-[400px] h-[400px] flex-col">
      <div v-for="y in 3" class="flex flex-col w-full h-1/3">
        <div class="h-[99%] w-full flex flex-row">
          <div v-for="x in 3" class="w-1/3 h-full flex flex-row">
            <div
              @click="round(x, y)"
              :class="`w-[99%] h-full flex justify-center items-center ${has_won || has_lost || !clickable ? 'pointer-events-none' : 'pointer-events-auto'}`"
            >
              <div
                v-if="player[y - 1][x - 1]"
                class="rounded-full border-2 w-[40px] h-[40px] border-dark_green"
              ></div>
              <div
                v-if="computer[y - 1][x - 1]"
                class="rounded-full border-2 w-[40px] h-[40px] bg-dark_green"
              ></div>
            </div>
            <div
              v-if="x == 1 || x == 2"
              class="border-2 border-dark_green h-full w-[1%]"
            ></div>
          </div>
        </div>
        <div
          v-if="y == 1 || y == 2"
          class="border-2 border-dark_green w-full h-[1%]"
        ></div>
      </div>
    </div>
    <div class="mt-8 w-full flex justify-center">
      <button @click="restart" class="border-dark_green border-2 rounded-full">
        <span class="text-2xl text-dark_green p-3">{{
          has_won || has_lost || tie ? "Play again" : "Restart Game"
        }}</span>
      </button>
    </div>
    <!-- test button -->
    <button
      @click="model_move"
      class="bg-dark_green w-[20px] h-[20px] mt-3"
    ></button>
  </div>
</template>
