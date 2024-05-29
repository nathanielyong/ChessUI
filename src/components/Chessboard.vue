<template>
  <div>
    <TheChessboard :player-color="playerColor" :board-config="boardConfig" reactive-config></TheChessboard>
    <div>
      {{ this.data }}
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
      username: "",
      error: "",
      boardConfig: reactive({
        coordinates: true,
        events: {
          move: (from, to, capture) => {
            this.makeMove(from, to)
          }
        }
      }),
      playerColor: '',
      data: {},
    };
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
    this.fetchGame();
  },
  methods: {
    async fetchGame() {
      try {
        const response = await axiosInstance.get(
          "/api/LiveChessGame/getCurrentGame"
        );
        console.log(response);
        this.data = response.data;
        const gameState = response.data.gameState;
        const isWhite = this.username === gameState.whitePlayerUsername

        this.boardConfig.fen = gameState.currentPositionFEN
        this.boardConfig.orientation = isWhite ? 'white' : 'black'
        this.playerColor = isWhite ? 'white' : 'black'
      } catch (err) {
        this.error = err;
      }
    },
    async makeMove(from, to) {
      const move = from + '-' + to;
      console.log(move)
      try {
        const response = await axiosInstance.post(
          `/api/LiveChessGame/makeMove?move=${move}`
        );
        console.log(response)
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
<style></style>