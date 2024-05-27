<template>
    <div>
        <TheChessboard :board-config="boardConfig" @board-created="(api) => (boardApi = api)"></TheChessboard>
        <div>
            {{ this.data }}
        </div>
    </div>
</template>
<script>
import { TheChessboard } from 'vue3-chessboard'
import 'vue3-chessboard/style.css'
import axiosInstance from '@/axiosConfig'
import { jwtDecode } from 'jwt-decode'

let boardApi;
const boardConfig = {
    coordinates: true,
    events: {
        move: (from, to, capture) => {

        }
    }
}
export default {
    data() {
        return {
            username: '',
            error: '',
            currentFen: "rnbqkbnr/pppppppp/8/8/4P3/8/PPPP1PPP/RNBQKBNR b KQkq e3 0 1",
            boardConfig: boardConfig,
            data: {}
        }
    },
    mounted() {
        const token = localStorage.getItem('jwtToken')
        if (token) {
            const decodedToken = jwtDecode(token);
            this.username = decodedToken["http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"]
        }
        this.fetchGame()
    },
    methods: {
        async fetchGame() {
            try {
                const response = await axiosInstance.get('/api/LiveChessGame/getCurrentGame')
                console.log(response)
                this.data = response.data
            } catch (err) {
                this.error = err
            }
        },
        async makeMove(from, to) {
            const move = from + to
            try {
                const response = await axiosInstance.get(`/api/LiveChessGame/makeMove?${move}`)
                console.log(response)
            } catch (err) {
                this.error = err
            }
        }
    },
    components: {
        TheChessboard
    }
}
</script>
<style></style>