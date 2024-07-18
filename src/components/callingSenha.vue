<template>
    <section>
        <div class="callingSenha-componenet">
         <div class="ContentCallingHeader">
            <h1>Chamando</h1>
         </div>   

        <div class="CalledCom">
            <div v-for="(called, index) in calledSenha" :key="index">
                    <div class="infoAtentende">
                        <h4>Senha</h4>
                        <h1>{{ called.senha }}</h1>
                        <h3>Guiche</h3>
                        <h1>{{ called.guiche }}</h1>
                        <h4>atendente</h4>
                        <h2>{{ called.atendente }}</h2>
                    </div>
            </div>
        </div>
            
         </div>
    
    </section>
   
   
</template>

<script>
import axios from 'axios';
export default {
    data() {
        return {
            calledSenha: []
        }
    },

    methods: {


        connectWebSocket() {
            const ws = new WebSocket('ws://localhost:8080');
            ws.onopen = () => {
                console.log('WebSocket connected on component call');
            };
            ws.onmessage = (event) => {
                const data = JSON.parse(event.data);

                console.log(data)

                if (data.status === 'sucess' && data.type === 'Call') {
                    console.log('called if triggred');
                    this.calledSenha.push(data.data);
                }

               else  if(data.status === 'success' && data.type === 'Delete'){  
                    this.calledSenha = this.calledSenha.filter(senha => senha._id !== data.data._id);
                }
            };
            ws.onerror = (error) => {
                console.error('WebSocket error:', error);
            };
            ws.onclose = () => {
                console.log('WebSocket closed');
            };
        },



        async getCurrentCalling(){
            try {
                const response = await axios.get('http://localhost:8080/currentCalling');
                this.calledSenha = response.data.data;
            } catch(error){
                console.error('Error calling senha', error);
            }
        }


    },

    mounted(){
        this.getCurrentCalling();
        this.connectWebSocket();
    }
}


</script>