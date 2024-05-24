<script setup lang="ts">
import { ref, defineProps, onMounted } from 'vue';

interface Player {
  name: string;
  symbol: 'X' | 'O';
}

const props = defineProps<{ players: Player[] }>();
const emit = defineEmits(['quitGame']);
const toggleModal = () => showModal.value = !showModal.value;

const currentPlayer = ref(props.players[0]);
const scores = ref({ X: 0, O: 0 });
const winner = ref<Player | null>(null);
const isDraw = ref(false);
const showModal = ref(false);

const gameBoard = ref([
  ['', '', ''],
  ['', '', ''],
  ['', '', ''],
]);

const gameHistory = ref<{ winner: Player | null, isDraw: boolean, score: number }[]>([]);

const winningLines = [
  [0, 1, 2],
  [3, 4, 5],
  [6, 7, 8],
  [0, 3, 6],
  [1, 4, 7],
  [2, 5, 8],
  [0, 4, 8],
  [2, 4, 6],
];

const getWinningPlayer = (squares: string[]): Player | null => {
  for (const [a, b, c] of winningLines) {
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return props.players.find(player => player.symbol === squares[a]) || null;
    }
  }
  return null;
};

const checkGameState = () => {
  const flatBoard = gameBoard.value.flat();
  winner.value = getWinningPlayer(flatBoard);

  if (winner.value) {
    scores.value[winner.value.symbol]++;
    updateScoreModal(winner.value);
  } else {
    isDraw.value = flatBoard.every(cell => cell !== '');
    if (isDraw.value) {
      gameHistory.value.push({ winner: null, isDraw: true, score: 0 });
    }
  }

  if (gameHistory.value.length > 5) {
    gameHistory.value.shift();
  }

  saveGame();
};

const makeAMove = (x: number, y: number) => {
  if (winner.value || isDraw.value || gameBoard.value[x][y]) return;
  gameBoard.value[x][y] = currentPlayer.value.symbol;
  currentPlayer.value = currentPlayer.value.symbol === 'X' ? props.players[1] : props.players[0];
  checkGameState();
};

const resetGame = () => {
  gameBoard.value = [
    ['', '', ''],
    ['', '', ''],
    ['', '', ''],
  ];
  currentPlayer.value = props.players[0];
  winner.value = null;
  isDraw.value = false;
  saveGame();
};

const updateScoreModal = (winner: Player) => {
  const winnerIndex = gameHistory.value.findIndex(game => game.winner?.name === winner.name);
  if (winnerIndex !== -1) {
    gameHistory.value[winnerIndex].score += 1;
  } else {
    gameHistory.value.push({ winner, isDraw: false, score: 1 });
  }
  gameHistory.value.sort((a, b) => b.score - a.score);
  localStorage.setItem('gameHistory', JSON.stringify(gameHistory.value));
};

const saveGame = () => {
  const gameState = {
    gameBoard: gameBoard.value,
    scores: scores.value,
    currentPlayer: currentPlayer.value,
    winner: winner.value,
    isDraw: isDraw.value
  };
  localStorage.setItem('ticTacToeState', JSON.stringify(gameState));
};

const loadGame = () => {
  const storedState = localStorage.getItem('ticTacToeState');
  if (storedState) {
    const gameState = JSON.parse(storedState);
    gameBoard.value = gameState.gameBoard;
    scores.value = gameState.scores;
    currentPlayer.value = gameState.currentPlayer;
    winner.value = gameState.winner;
    isDraw.value = gameState.isDraw;
  }

  const storedHistory = localStorage.getItem('gameHistory');
  if (storedHistory) {
    gameHistory.value = JSON.parse(storedHistory);
  }
};

const resetScore = () => {
  gameHistory.value = []
  localStorage.setItem('scores', JSON.stringify(gameHistory.value));
}

onMounted(() => {
  loadGame();
});
</script>

<template>
  <div class="mainContainer">
    <div class="fontContainer">
      <h1>Tic Tac Toe</h1>
      <h2 class="score">{{ scores.O }} - {{ scores.X }}</h2>
      <h2 v-if="winner" class="winningText">Player '{{ winner.name }} ({{ winner.symbol }})' wins!</h2>
      <h2 class="drawTxt" v-else-if="isDraw">Draw!</h2>
      <h3 class="currentPlayer" v-if="!winner">Player {{ currentPlayer.name }} ({{ currentPlayer.symbol }})'s turn</h3>
    </div>

    <div class="gridContainer">
      <div v-for="(row, x) in gameBoard" :key="x" class="flexContainer">
        <div v-for="(cell, y) in row" :key="y" @click="makeAMove(x, y)" class="flexChild">
          {{ cell }}
        </div>
      </div>
    </div>

    <div class="btnContainer">
      <button class="retryGameBtn" @click="resetGame">Retry</button>
      <button class="showHistoryBtn"@click="toggleModal">Score History</button>
      <button class="quitBtn" @click="$emit('quitGame')">Quit</button>
    </div>
      
    <div v-if="showModal" class="modalOverlay">
      <div class="modalContent">
        <h2>Score History</h2>
        <p v-for="(game, index) in gameHistory" :key="index">
          <template v-if="!game.isDraw">
            <span><span class="winnerTxt">Winner:</span> {{ game.winner?.name }} ({{ game.winner?.symbol }}) <span class="scoreTxt">Score:</span> {{ game.score }}</span>
          </template>
        </p>
        <button class="resetScoreBtn"@click="resetScore">Reset</button>
        <button class="closeModalBtn"@click="toggleModal">Close</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import 'src/sass/gameLogic.scss';
@import 'src/sass/modalGameLogic.scss';
</style>
