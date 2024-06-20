<template>
  <div>
    <div class="chess-container">
      <TheChessboard :style="{ margin: '0' }" @move="makeMove" :player-color="playerColor" :board-config="boardConfig" reactive-config>
      </TheChessboard>
      <div class="timers">
        <div>{{ this.isPlayerWhite ? this.blackPlayerUsername : this.whitePlayerUsername }}</div>
        <div class="timer">{{ this.isPlayerWhite ? formattedBlackTime : formattedWhiteTime }}</div>
        <div>{{ this.result !== "" ? this.gameEndReason + " " + this.result : "" }}</div>
        <div class="timer">{{ this.isPlayerWhite ? formattedWhiteTime : formattedBlackTime }}</div>
        <div>{{ this.isPlayerWhite ? this.whitePlayerUsername : this.blackPlayerUsername }}</div>
      </div>
    </div>
  </div>
</template>
<script>
import { reactive } from "vue";
import { TheChessboard } from "vue3-chessboard";
import "vue3-chessboard/style.css";
import axiosInstance from "@/axiosConfig";
import { jwtDecode } from "jwt-decode";

export default {
  data() {
    return {
      intervalId: null,
      whitePlayerUsername: "",
      blackPlayerUsername: "",
      isPlayerWhite: true,
      isWhiteTurn: true,
      whiteTime: 0,
      blackTime: 0,
      moveCount: 0,
      result: "",
      gameEndReason: "",
      error: "",
      boardConfig: reactive({
        coordinates: true
      }),
      playerColor: '',
      data: {},
    };
  },
  computed: {
    formattedWhiteTime() {
      const minutes = Math.max(0, Math.floor(this.whiteTime / 60000)).toString().padStart(2, '0')
      const seconds = Math.max(0, Math.floor(this.whiteTime % 60000 / 1000)).toString().padStart(2, '0')
      return `${minutes}:${seconds}`
    },
    formattedBlackTime() {
      const minutes = Math.max(0, Math.floor(this.blackTime / 60000)).toString().padStart(2, '0')
      const seconds = Math.max(0, Math.floor(this.blackTime % 60000 / 1000)).toString().padStart(2, '0')
      return `${minutes}:${seconds}`
    }
  },
  mounted() {
    const token = localStorage.getItem("jwtToken");
    if (token) {
      const decodedToken = jwtDecode(token);
      this.username =
        decodedToken[
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"
        ];
    }
    this.fetchGame()
    this.startTimer()
  },
  methods: {
    startTimer() {
      if (this.intervalId) {
        clearInterval(this.intervalId)
      }
      this.intervalId = setInterval(() => {
        if (this.moveCount > 1 && this.result === "") {
          if (this.isWhiteTurn) {
            if (this.whiteTime > 0) {
              this.whiteTime -= 1000
            } else {
              clearInterval(this.intervalId)
            }
          } else {
            if (this.blackTime > 0) {
              this.blackTime -= 1000
            } else {
              clearInterval(this.intervalId)
            }
          }
        }
      }, 1000)
    },
    async fetchGame() {
      try {
        const response = await axiosInstance.get(
          "/api/LiveChessGame/getCurrentGame"
        );
        console.log(response);
        this.data = response.data;
        const gameState = response.data.gameState;
        this.whitePlayerUsername = gameState.whitePlayerUsername
        this.blackPlayerUsername = gameState.blackPlayerUsername
        this.isWhiteTurn = gameState.isWhiteTurn
        this.isPlayerWhite = this.username === this.whitePlayerUsername
        this.whiteTime = gameState.whiteTimeRemaining
        this.blackTime = gameState.blackTimeRemaining
        this.moveCount = gameState.moveCount
        this.result = gameState.result
        this.gameEndReason = gameState.gameEndReason
        this.boardConfig.fen = gameState.currentPositionFEN
        this.boardConfig.orientation = this.isPlayerWhite ? 'white' : 'black'
        this.boardConfig.viewOnly = this.result !== ""
        this.playerColor = this.isPlayerWhite ? 'white' : 'black'
      } catch (err) {
        this.error = err;
      }
    },
    async makeMove(move) {
      const moveString = move.from + '-' + move.to;
      console.log(moveString)
      try {
        const response = await axiosInstance.post(
          `/api/LiveChessGame/makeMove?move=${moveString}`
        )
        console.log(response)
        this.data = response.data
        const gameState = response.data.gameState
        this.isWhiteTurn = gameState.isWhiteTurn
        this.whiteTime = gameState.whiteTimeRemaining
        this.blackTime = gameState.blackTimeRemaining
        this.moveCount = gameState.moveCount
        this.result = gameState.result
        this.gameEndReason = gameState.gameEndReason
        this.boardConfig.viewOnly = true
      } catch (err) {
        this.error = err;
      }
    },

  },
  components: {
    TheChessboard,
  },
};
</script>
<style scoped>
.chess-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 30px;
}

.timers {
  width: 200px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 20px;
}

.timer {
  background-color: #f0f0f0;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  text-align: center;
  width: 100px;
}
</style>