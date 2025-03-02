<template>
    <v-app>
        <v-main>
            <WeatherPanel />
        </v-main>
    </v-app>
</template>

<script>
import WeatherPanel from './components/WeatherPanel.vue';

export default {
    components: {
        WeatherPanel,
    },
};
</script>

<style></style>
