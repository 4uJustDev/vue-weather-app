<template>
    <v-container>
        <v-card>
            <v-card-title class="headline">Погода</v-card-title>
            <v-card-text>
                <v-combobox
                    clearable
                    variant="underlined"
                    label="Введите город"
                    :items="cities"
                    @update:search="fetchCitySuggestions"
                    @keyup.enter="fetchWeather"
                    v-model="city"
                ></v-combobox>

                <v-btn color="primary" @click="fetchWeather">
                    Получить погоду
                </v-btn>

                <v-divider class="my-4"></v-divider>

                <div v-if="weather">
                    <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>
                    <p>{{ weather.weather[0].description }}</p>
                    <p>
                        Температура:
                        {{ Math.round(weather.main.temp) }}°C
                    </p>
                    <p>Влажность: {{ weather.main.humidity }}%</p>
                    <p>Ветер: {{ weather.wind.speed }} м/с</p>
                </div>
            </v-card-text>
        </v-card>

        <!-- Уведомление об ошибке -->
        <v-snackbar v-model="showError" color="error" timeout="3000">
            {{ errorMessage }}
            <template v-slot:action="{ attrs }">
                <v-btn text v-bind="attrs" @click="showError = false">
                    Закрыть
                </v-btn>
            </template>
        </v-snackbar>
    </v-container>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            city: '',
            cities: [],
            weather: null,
            apiWeatherKey: process.env.VUE_APP_WEATHER_API_KEY,

            // Уведомления об ошибках
            showError: false,
            errorMessage: '',
        };
    },

    created() {
        this.checkEnv();
    },

    methods: {
        checkEnv() {
            if (!this.apiWeatherKey) {
                this.errorMessage =
                    'Ошибка: Отсутствует API ключ <VUE_APP_OPENWEATHERMAP_API_KEY>. Проверьте файл .env';
                this.showError = true;
                return false;
            }
            return true;
        },

        async fetchCitySuggestions(query) {
            if (!this.checkEnv() || !query) {
                this.cities = [];
                return;
            }

            try {
                const response = await axios.get(
                    `https://api.openweathermap.org/geo/1.0/direct?q=${query}&limit=5&appid=${this.apiWeatherKey}`,
                );

                this.cities = response.data.map((city) => city.name);
                console.log(this.cities);
            } catch (error) {
                console.error('Ошибка при поиске городов:', error);
                this.errorMessage = 'Не удалось загрузить список городов';
                this.showError = true;
            }
        },

        async fetchWeather() {
            if (!this.checkEnv() || !this.city) {
                return;
            }

            try {
                const response = await axios.get(
                    `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=${this.apiWeatherKey}`,
                );

                this.weather = response.data;
            } catch (error) {
                console.error('Ошибка при получении данных о погоде:', error);
                this.errorMessage = 'Не удалось получить данные о погоде';
                this.showError = true;
            }
        },
    },
};
</script>
