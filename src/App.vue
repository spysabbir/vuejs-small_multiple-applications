<script setup>

</script>

<template>
    <div class="container mx-auto bg-gray-400 p-5 rounded mt-5">
        <h1 class="text-4xl text-center">Hello Vue 3</h1>
        <p class="text-center">This is a simple vue 3 project</p>
        <div>
            <hr class="my-5">
            <div class="flex justify-center">
                <router-link to="/bmi-calculator" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Bmi Calculator</router-link>
                <router-link to="/calculator" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Calculator</router-link>
                <router-link to="/currency-converter" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Currency Converter</router-link>
                <router-link to="/stopwatch" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Stopwatch</router-link>
                <router-link to="/url-shortener" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Url Shortener</router-link>
                <router-link to="/weather" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Weather</router-link>
                <router-link to="/countdown" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Countdown</router-link>
                <router-link to="/media-player" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Media Player</router-link>
                <router-link to="/photo-gallery" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Photo Gallery</router-link>
                <router-link to="/quiz" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Quiz</router-link>
                <router-link to="/todo" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mx-2">Todo</router-link>
            </div>
            <hr class="my-5">
            <div>
                <router-view></router-view>
            </div>
        </div>
    </div>
</template>

<style scoped>
.router-link-active {
    background-color: #3aaa07;
    color: white;
}
</style>
