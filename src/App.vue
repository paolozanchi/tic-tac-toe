<template>
  <div id="app">
    <button
      v-if="gameEnded"
      @click="newGame"
    >
      New Game
    </button>
    <p>Winner: {{ winner }}</p>
    <TicTacToeBoard
      :board-state="board"
      @clickedCell="clickedCell"
    />
  </div>
</template>

<script>
  import TicTacToeBoard from '@/components/TicTacToeBoard.vue'

  export default {
    name: 'App',
    components: {
      TicTacToeBoard
    },
    data() {
      return {
        ai: 'O',
        player: 'X',
        board: [],
        gameEnded: false,
        isPlayerTurn: true,
        winner: null,
        scores: {
          X: 1,
          O: -1,
          tie: 0
        }
      }
    },
    mounted() {
      this.newGame()
    },
    methods: {
      newGame() {
        // TODO: cambiare
        this.isPlayerTurn = false
        this.ai= 'X'
        this.player = 'O'

        this.gameEnded = false
        this.winner = null
        this.board = [];
        for (let i = 0; i < 3; i++) {
          this.board[i] = [null, null, null]
        }

        if (!this.isPlayerTurn) 
          this.makeAIMove()
      },
      clickedCell(rowIndex, cellIndex) {
        // Clicked cell is already occupied
        if (this.board[rowIndex - 1][cellIndex - 1] != null) return

        // The game already ended or it's not the player's turn
        if (this.gameEnded || !this.isPlayerTurn) return
        
        this.fillBoardCell(this.board, rowIndex - 1, cellIndex - 1, this.player)
        this.isXturn = !this.isXturn
        this.isPlayerTurn = false
        this.winner = this.checkWinner(this.board)

        if (this.winner != null) {
          this.gameEnded = true
          return
        }

        this.makeAIMove()
      },
      fillBoardCell(board, rowIndex, cellIndex, playerSign) {
        let rowArray = board[rowIndex]
        this.$set(rowArray, cellIndex, playerSign)
        this.$set(board, rowIndex, rowArray)
      },
      clearBoardCell(board, rowIndex, cellIndex) {
        let rowArray = board[rowIndex]
        this.$set(rowArray, cellIndex, null)
        this.$set(board, rowIndex, rowArray)
      },
      checkWinner(board) {
        // Horizontal
        for (let i = 0; i < 3; i++) {
          if (board[i][0] != null && (board[i][0] == board[i][1] && board[i][0] == board[i][2])) {
            return board[i][0]
          }
        }

        // Vertical
        for (let i = 0; i < 3; i++) {
          if (board[0][i] != null && (board[0][i] == board[1][i] && board[0][i] == board[2][i])) {
            return board[0][i]
          }
        }

        // Diagonals
        if (board[0][0] != null && (board[0][0] == board[1][1] && board[0][0] == board[2][2]))
          return board[0][0]
          
        if (board[0][2] != null && (board[0][2] == board[1][1] && board[0][2] == board[2][0]))
          return board[0][2]

        // Tie
        let availableCells = 0
        for(let i = 0; i < 3; i++) {
          for(let j = 0; j < 3; j++) {
            // If the cell is available
            if(this.board[i][j] == null) {
              availableCells++
            }
          }
        }

        if (availableCells == 0)
          return 'tie'

        // No win
        return null
      },
      makeAIMove() {
        console.time("makeAIMove")
        let bestScore = -Infinity
        let bestMove = null

        for(let i = 0; i < 3; i++) {
          for(let j = 0; j < 3; j++) {
            // If the cell is available
            if(this.board[i][j] == null) {
              // Place the AI mark
              this.fillBoardCell(this.board, i, j, this.ai)

              // Calculate the score for the new position
              let newScore = this.miniMax(this.board, false)

              // Undo the AI move
              this.clearBoardCell(this.board, i, j)
              
              if (newScore > bestScore) {
                bestScore = newScore
                bestMove = { i, j }
              }
            }
          }
        }
        
        this.fillBoardCell(this.board, bestMove.i, bestMove.j, this.ai)
        this.winner = this.checkWinner(this.board)

        if (this.winner != null) {
          this.gameEnded = true
        }
        else {
          this.isPlayerTurn = true
        }
        console.timeEnd("makeAIMove")
      },
      miniMax(board, isMaximazing) {
        let result = this.checkWinner(board)

        // Match ended, return the result
        if (result != null) {
          return this.scores[result]
        }

        if (isMaximazing) {
          let bestScore = -Infinity

          for (let i = 0; i < 3; i++) {
            for (let j = 0; j < 3; j++) {
              // If the cell is available
              if (this.board[i][j] == null) {
                // Place the AI mark
                this.fillBoardCell(this.board, i, j, this.ai)

                // Calculate the score for the new position
                let newScore = this.miniMax(this.board, false)

                // Undo the AI move
                this.clearBoardCell(this.board, i, j)
                
                // Check if the position reached is better
                bestScore = Math.max(newScore, bestScore)
              }
            }
          }

          return bestScore
        }
        else {
          let bestScore = +Infinity

          for (let i = 0; i < 3; i++) {
            for (let j = 0; j < 3; j++) {
              // If the cell is available
              if (this.board[i][j] == null) {
                // Place the player's mark
                this.fillBoardCell(this.board, i, j, this.player)

                // Calculate the score for the new position
                let newScore = this.miniMax(this.board, true)

                // Undo the AI move
                this.clearBoardCell(this.board, i, j)
                
                // Check if the position reached is better
                bestScore = Math.min(newScore, bestScore)
              }
            }
          }

          return bestScore
        }
      }
    }
  }
</script>

<style>
  :root {
    --background-color: #121212;
    --lighter-color: #352b41;
    --accent-color: #d3b0fd;
  }

  html, body {
    background-color: var(--background-color);
  }

  #app {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Helvetica Neue", Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    color: var(--accent-color);
  }
</style>