<script setup lang="ts">
import { ref, onMounted } from 'vue';
import AddPlayer from './AddPlayer.vue';
import GameLogic from './GameLogic.vue';

interface Player {
  name: string;
  symbol: 'X' | 'O';
}

const playerList = ref<Player[]>([]);
const hideGame = ref<boolean>(false)

const addPlayer = (playerName: string) => {
    if(playerList.value.length < 2) {
        const symbol = playerList.value.length === 0 ? 'X' : 'O'; 
        playerList.value.push({ name: playerName, symbol: symbol })
    } else {
        alert('Maximal antal spelare')
    }

    localStorage.setItem('players', JSON.stringify(playerList.value));
}

const startGameBtn  = () => {
    hideGame.value = true;
}

const quitGame = () => {
    hideGame.value = false;
    playerList.value = [];
    localStorage.removeItem('players');
    localStorage.removeItem('ticTacToeState');
}

onMounted(() => {
    const storedPlayers = localStorage.getItem('players');
    if (storedPlayers) {
        playerList.value = JSON.parse(storedPlayers);
        hideGame.value = true;
    }
});

</script>

<template>
    <div>
        <AddPlayer @add-player="addPlayer" @start-btn="startGameBtn" v-if="!hideGame" />
        <div v-if="!hideGame">
            <h2> Spelare:</h2>
            <div class="playerNameContainer">
                <h4 v-for="(player, index) in playerList" :key="index"> {{ player.name }} - {{ player.symbol }} </h4>
            </div>
        </div>
        
        <div>
            <GameLogic v-if="hideGame" :players="playerList" @quit-game="quitGame" />
        </div>
    </div>
</template>

<style lang="scss" scoped>
@import 'src/sass/parent.scss';
</style>
