<script setup lang="ts">
const WIDTH = 10;
const HEIGHT = 10;

interface BlockState {
  x?: number;
  y?: number;
  revealed: boolean;
  mine?: boolean;
  flagged?: boolean;
  adjacentMines: number;
}

const state = reactive(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from(
      { length: WIDTH },
      (_, x): BlockState => ({
        x,
        y,
        adjacentMines: 0,
        revealed: false,
      })
    )
  )
);

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [0, 1],
  [-1, 0],
  [-1, 1],
  [-1, -1],
];

const numberColors = [
  "text-transparent",
  "text-blue-500",
  "text-green-500",
  "text-yellow-500",
  "text-orange-500",
  "text-red-500",
  "text-purple-500",
  "text-pink-500",
  "text-teal-500",
];

let mineGenerated = false;
const dev = true;

function generateMines(initial: BlockState) {
  for (const row of state) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) < 1) continue;
      if (Math.abs(initial.y - block.y) < 1) continue;
      block.mine = Math.random() < 0.2;
    }
  }
}

function getSiblings(block: BlockState) {
  return directions
    .map(([dx, dy]) => {
      const x1 = block.x + dx;
      const y1 = block.y + dy;
      if (x1 >= WIDTH || x1 < 0 || y1 >= HEIGHT || y1 < 0) return undefined;
      return state[x1][y1];
    })
    .filter(Boolean) as BlockState[];
}

function updateNumbers() {
  state.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine) return;
      getSiblings(block).forEach((sibling) => {
        if (sibling.mine) block.adjacentMines += 1;
      });
    });
  });
}

function getBlockClass(block: BlockState) {
  if (!block.revealed) return "bg-gray-500/40";
  return block.mine ? "bg-red-500/50" : numberColors[block.adjacentMines];
}

function expandZero(block: BlockState) {
  if (block.adjacentMines) return;
  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true;
      expandZero(s);
    }
  });
}

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block);
    updateNumbers();
    mineGenerated = true;
  }
  if (block.mine) alert("BOOOM!");
  block.revealed = true;
  expandZero(block);
}
</script>

<template>
  <div>
    Minesweeper
    <div p-5>
      <div
        v-for="(row, y) in state"
        :key="y"
        flex="~"
        items-center
        justify-center
      >
        <button
          v-for="(block, x) in row"
          flex="~"
          items-center
          justify-center
          :key="x"
          w-8
          h-8
          hover="bg-gray/10"
          border="1 gray-400/30"
          m="0.5"
          :class="getBlockClass(block)"
          @click="onClick(block)"
        >
          <template v-if="block.revealed || dev">
            <div v-if="block.mine" i-mdi-mine></div>
            <div v-else>
              {{ block.adjacentMines }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
