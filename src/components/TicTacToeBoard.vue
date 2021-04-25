<template>
  <table class="board">
    <tr
      v-for="rowIndex in 3"
      :key="rowIndex"
    >
      <td
        v-for="cellIndex in 3"
        :key="rowIndex * cellIndex"
        class="cell"
        @click="clickedCell(rowIndex, cellIndex)"
      >
        <span
          v-if="boardState[rowIndex - 1]"
          class="cellContent"
        >
          {{ boardState[rowIndex - 1][cellIndex - 1] }}
        </span>
      </td>
    </tr>
  </table>
</template>

<script>
  export default {
    name: 'TicTacToeBoard',
    props: {
      boardState: {
        type: Array,
        required: true
      }
    },
    methods: {
      clickedCell(rowIndex, cellIndex) {
        this.$emit("clickedCell", rowIndex, cellIndex);
      }
    }
  }
</script>

<style scoped>
  .board {
    margin: 0 auto;
    border-collapse: collapse;
  }
  
  .cell {
    border: 3px solid var(--accent-color);
    height: 200px;
    width: 200px;
  }

  .cell:hover {
    background-color: var(--lighter-color);
  }

  .cellContent {
    font-size: 42pt;
    font-weight: bolder;
    user-select: none;
  }
</style>
