<template>
  <section
    id="app"
    class="container"
  >
    <main>
      <RippleButton
        variant="primary"
        @click="newGame"
      >
        New Game
      </RippleButton>

      <TicTacToeBoard
        :board-state="board"
        @clickedCell="clickedCell"
      />

      <h1 v-if="winner">
        Winner: {{ winner }}
      </h1>
    </main>
    
    <footer>
      <span v-if="appVersion">
        v{{ appVersion }} - 
      </span>

      Made with <span class="heart">❤️</span> by 
      <a 
        href="https://github.com/paolozanchi"
        class="link"
        target="_github"
      >mmmmmeh1</a>
      |
      <a
        href="https://linkedin.com/in/paolo-zanchi"
        class="link"
        target="_linkedin"
      >
        Paolo Zanchi
      </a>
    </footer>
  </section>
</template>

<script>
  import TicTacToeBoard from '@/components/TicTacToeBoard.vue'
  import RippleButton from '@/components/RippleButton.vue'
  import { version } from '../package.json'

  export default {
    name: 'App',
    components: {
      TicTacToeBoard,
      RippleButton,
    },
    data() {
      return {
        appVersion: version,
        board: [],
        checkedPositions: 0,
        gameEnded: false,
        isPlayerTurn: true,
        maximizingPlayer: null,
        minimizingPlayer: null,
        playerAIMark: null,
        playerHumanMark: null,
        scores: {
          X: 10,
          O: -10,
          tie: 0
        },
        winner: null,
      }
    },
    mounted() {
      this.newGame()
    },
    methods: {
      newGame() {
        if (this.playerAIMark === 'X') {
          this.isPlayerTurn = true
          this.playerAIMark = 'O'
          this.playerHumanMark = 'X'
          this.maximizingPlayer = this.playerHumanMark
          this.minimizingPlayer = this.playerAIMark
        } else {
          this.isPlayerTurn = false
          this.playerAIMark = 'X'
          this.playerHumanMark = 'O'
          this.maximizingPlayer = this.playerAIMark
          this.minimizingPlayer = this.playerHumanMark
        }

        this.gameEnded = false
        this.winner = null
        this.board = []
        for (let i = 0; i < 3; i++) {
          this.board[i] = [null, null, null]
        }

        if (!this.isPlayerTurn) 
          this.makeAIMove()
      },
      async clickedCell(rowIndex, cellIndex) {
        // Clicked cell is already occupied
        if (this.board[rowIndex - 1][cellIndex - 1] != null) return

        // The game already ended or it's not the player's turn
        if (this.gameEnded || !this.isPlayerTurn) return
        
        this.makeMove(this.board, rowIndex - 1, cellIndex - 1, this.playerHumanMark)
        this.isPlayerTurn = false
        this.winner = this.checkWinner(this.board)

        if (this.winner != null) {
          this.gameEnded = true
          return
        }

        await this.sleep(500)

        this.makeAIMove()
      },
      makeMove(board, rowIndex, cellIndex, mark) {
        let rowArray = board[rowIndex]
        this.$set(rowArray, cellIndex, mark)
        this.$set(board, rowIndex, rowArray)
      },
      undoMove(board, rowIndex, cellIndex) {
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
        if (this.getAvailableCellsNumber(board) == 0)
          return 'tie'

        // No win
        return null
      },
      getAvailableCellsNumber(board) {
        let availableCells = 0
        for(let i = 0; i < 3; i++) {
          for(let j = 0; j < 3; j++) {
            // If the cell is available
            if(board[i][j] == null) {
              availableCells++
            }
          }
        }

        return availableCells
      },
      makeAIMove() {
        console.time("makeAIMove")
        const MINIMAXDEPTH = 9
        const AIisMaximazing = (this.playerAIMark == this.maximizingPlayer)
        let bestScore = AIisMaximazing ? -Infinity : +Infinity
        let bestMove = null
        this.checkedPositions = 0

        loop: for(let i = 0; i < 3; i++) {
          for(let j = 0; j < 3; j++) {
            // If the cell is available
            if(this.board[i][j] == null) {
              // Place the AI mark
              this.makeMove(this.board, i, j, this.playerAIMark)

              // Calculate the score for the new position
              let newScore = this.miniMaxAlphaBeta(this.board, MINIMAXDEPTH, -Infinity, +Infinity, !AIisMaximazing)

              // Undo the AI move
              this.undoMove(this.board, i, j)
              
              if (AIisMaximazing) {
                if (newScore > bestScore) {
                  bestScore = newScore
                  bestMove = { i, j }
                }
              } else {
                if (newScore < bestScore) {
                  bestScore = newScore
                  bestMove = { i, j }
                }
              }

              if(bestScore == this.scores[this.playerAIMark]) {
                // Found a win, don't check further
                break loop
              }
            }
          }
        }
        
        this.makeMove(this.board, bestMove.i, bestMove.j, this.playerAIMark)
        this.winner = this.checkWinner(this.board)

        if (this.winner != null) {
          this.gameEnded = true
        }
        else {
          this.isPlayerTurn = true
        }
        console.timeEnd("makeAIMove")
        console.debug("checked", this.checkedPositions, "positions with depth", MINIMAXDEPTH)
      },
      miniMaxAlphaBeta(board, depth, alpha, beta, isMaximazing) {
        let result = this.checkWinner(board)

        if (result != null) {
          // The match ended (win/lost/tie), return the result
          return this.scores[result]
        }
        else if (depth === 0) {
          // TODO: The match did not end, who's winning?
          return 0
        }
        
        this.checkedPositions++
        
        let bestScore = isMaximazing? -Infinity : +Infinity

        for (let i = 0; i < 3; i++) {
          for (let j = 0; j < 3; j++) {
            // If the cell is available
            if (board[i][j] == null) {
              this.makeMove(board, i, j, isMaximazing ? this.maximizingPlayer : this.minimizingPlayer)
              
              // Calculate the score for the new position
              let newScore = this.miniMaxAlphaBeta(board, depth - 1, alpha, beta, !isMaximazing)

              this.undoMove(board, i, j)
                
              // Check if the reached position is better than the current best found
              bestScore = isMaximazing ? Math.max(newScore, bestScore) : Math.min(newScore, bestScore)
              
              // Alpha-Beta pruning
              if(isMaximazing) {
                alpha = Math.max(alpha, bestScore)
                if(alpha >= beta) break
              }
              else {
                beta = Math.min(beta, bestScore)
                if(beta <= alpha) break
              }
            }
          }
        }

        return bestScore
      },
      sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms))
      }
    }
  }
</script>

<style>
  :root {
    --dark: #121212;
    --light: #eee;
    --lighter-accent: #352b41;
    --accent: #d3b0fd;
    --rippleRgb: rgb(238, 238, 238);
  }

  html, body {
    background-color: var(--dark);
    height: 100%;
    margin: 0;
  }

  body {
    display: flex;
    flex-direction: column;
  }
  
  #app {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Helvetica Neue", 
    Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    margin-top: 60px;
    color: var(--light);
  }

  .container {
    flex: 1 0 auto;
    display: flex;
    flex-direction: column;
  }
  
  main {
    flex: 1 0 auto;
  }

  footer {
    flex-shrink: 0;
    padding: .5rem;
    opacity: .8;
  }
  
  .link {
    color: var(--accent);
  }

  .heart {
    color: transparent;
    text-shadow: 0 0 0 var(--accent);
  }
</style>