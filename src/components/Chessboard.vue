<template>
  <div>
    <TheChessboard :board-config="boardConfig" reactive-config></TheChessboard>
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
      boardConfig: reactive({}),
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
        const newBoardConfig = {
          fen: gameState.fen,
          orientation: isWhite ? 'white' : 'black',
          viewOnly: (gameState.isWhiteTurn && !isWhite) || (!gameState.isWhiteTurn && isWhite)
        }
        Object.assign(this.boardConfig, newBoardConfig)

      } catch (err) {
        this.error = err;
      }
    },
    async makeMove(from, to) {
      const move = from + to;
      try {
        const response = await axiosInstance.get(
          `/api/LiveChessGame/makeMove?${move}`
        );
        console.log(response);
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