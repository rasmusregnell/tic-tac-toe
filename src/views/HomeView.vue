<script setup lang="ts">
import { ref } from "vue";

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

function check_has_won(board: boolean[][]): boolean {
  // Check rows
  for (let i = 0; i < 3; i++) {
    if (board[i][0] === true && board[i][1] === true && board[i][2] === true) {
      return true;
    }
  }

  // Check columns
  for (let j = 0; j < 3; j++) {
    if (board[0][j] === true && board[1][j] === true && board[2][j] === true) {
      return true;
    }
  }

  // Check diagonals
  if (board[0][0] === true && board[1][1] === true && board[2][2] === true) {
    return true;
  }
  if (board[0][2] === true && board[1][1] === true && board[2][0] === true) {
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

function computer_move() {
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
  //player makes move
  if (!get_possible_moves().find((e) => arraysEqual(e, [y - 1, x - 1]))) {
    console.log("hej");
    return;
  }
  //console.log(get_possible_moves());
  player.value[y - 1][x - 1] = true;
  if (check_condition(player.value, true)) {
    return;
  }
  clickable.value = false;
  setTimeout(() => {
    computer_move();
    check_condition(computer.value, false);
    clickable.value = true;
  }, 1000);
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
      <span v-if="has_won" class="text-dark_green text-2xl">You Won!</span>
    </div>
    <div class="w-full flex justify-center">
      <span v-if="tie" class="text-dark_green text-2xl">It is a tie!</span>
    </div>
    <div class="w-full flex justify-center">
      <span v-if="has_lost" class="text-[#ff0000] text-2xl">You Lost!</span>
    </div>
    <div class="w-[400px] h-[400px] bg-[#FFFFFF] flex-col">
      <div v-for="y in 3" class="flex flex-row w-full h-1/3">
        <div
          v-for="x in 3"
          @click="round(x, y)"
          :class="`w-1/3 h-full border-[#000000] border-2 flex justify-center items-center ${has_won || has_lost || !clickable ? 'pointer-events-none' : 'pointer-events-auto'}`"
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
      </div>
    </div>
    <div class="w-full flex justify-center">
      <button
        v-if="has_won || has_lost || tie"
        @click="restart"
        class="text-2xl text-dark_green"
      >
        Play again
      </button>
    </div>
  </div>
</template>
