<template>
    <section>
        <div class="topContainer">
                <userOn></userOn>
            <div class="hora">
                <p>2</p>
            </div>
        </div>

        <div class="keysContainer">
            <h1 class="top">Fila de Espera</h1>
            <div class="keyContent">
                <div v-for="senha in senhas" :key="senha._id" class="key">
                    <p class="topIndicator">Senha</p>
                    <p class="Number">{{ senha.senha }}</p>
                </div>
            </div>
        </div>
    </section>

    <section>
        <prefComponent></prefComponent>
    </section>
</template>

<script>

import axios from 'axios';
import prefComponent from '../components/pref_senhas.vue';
import userOn from '../components/guicheCompnenet.vue';

export default {
    data() {
        return {
            senhas: [],
        };
    },

    methods: {

        connectWebSocket() {
            const ws = new WebSocket('ws://localhost:8080');
            ws.onopen = () => {
                console.log('WebSocket connected');
            };
            ws.onmessage = (event) => {
                const data = JSON.parse(event.data);
                if (data.status === 'success' && data.type === 'Regular') {
                    this.senhas.push(data.data);
                }

               if(data.status === 'success' && data.type === 'DeleteRegular'){
                    this.senhas = this.senhas.filter(senha => senha._id !== data.data._id);
                } 
            };
            ws.onerror = (error) => {
                console.error('WebSocket error:', error);
            };
            ws.onclose = () => {
                console.log('WebSocket closed');
            };
        },

        async fetchSenhas() {
            try {
                const response = await axios.get('http://localhost:8080/senhas');
                this.senhas = response.data.data;
            } catch (error) {
                console.error('Error fetching senhas:', error.response ? error.response.data : error.message);
            }
        },
    },

    mounted() {
        this.fetchSenhas();
        this.connectWebSocket();
    },

    components: {
        prefComponent,
        userOn
    }

};
</script>