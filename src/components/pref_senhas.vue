<template>
   <div class="PrefQuery" :style="{display: displayPref ? 'block' : 'none'}">
        <div class="PrefContainer">
            <div class="TopIndi">
                <h1 class="topPref">Fila de Espera Preferencial</h1>
            </div>
          
            <div class="PrefInline">
                <div class="prefInlineContent">
                    <div v-for="senha in senhas" :key="senha._id" class="prefKeys">
                            <p class="topIndiPref">Senha</p>
                            <p class="prefNumber">{{ senha.senha }}</p>                  
                    </div>
                </div>
            </div>

        </div>
   </div>
</template>

<script>
import axios from 'axios';
export default{
    data(){
        return {
            senhas: [],
            displayPref: 'none'
        }
    },


    computed: {
        displayPref() {
            return this.senhas.length > 0;
        }
    },


    methods: {

        connectWebSocket() {
            const ws = new WebSocket('ws://localhost:8080');
            ws.onopen = () => {
                console.log('WebSocket connected');
            };
            ws.onmessage = (event) => {
                const data = JSON.parse(event.data);
                if (data.status === 'success' && data.type === 'Preferencial') {
                    this.senhas.push(data.data);
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
                const response = await axios.get('http://localhost:8080/prefe');
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

   
}
</script>