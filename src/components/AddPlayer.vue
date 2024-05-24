<script setup lang="ts">
import { defineEmits, ref } from 'vue';

const inputValue = ref<string>('')
const emit = defineEmits(['addPlayer','startBtn']);

const currentPlayer = ref<string>('X');

const hideGame = ref<boolean>(true)
const startBtn = ref<boolean>(false)
const registerBtn = ref<boolean>(true)


const addingPlayer = () => {
    if(inputValue.value.trim() !== '') {
        emit('addPlayer', inputValue.value);
        currentPlayer.value = currentPlayer.value === "X" ? "O" : "✅" ;
        showAndHideBtn()
        inputValue.value = "";
    }
}

const startGameBtn = () => {
 emit('startBtn', hideGame.value);
}
const showAndHideBtn = () => {
    if(currentPlayer.value === "✅") {
        startBtn.value = true;
        registerBtn.value = false;
    } 
}

</script>

<template>
    <div class="formContainer">
        <h1>Registrera spelare {{ currentPlayer }} </h1>
        <form @submit.prevent="addingPlayer">
            <input v-model="inputValue" type="text" placeholder="Ange namn" maxlength="10">
            <button v-show="registerBtn" type="submit">Registrera</button>
            <button v-show="startBtn" @click="startGameBtn">Starta Spelet</button>
        </form>
    </div>
</template>

<style lang="scss" scoped>
@import 'src/sass/addPlayer.scss';
    
</style>