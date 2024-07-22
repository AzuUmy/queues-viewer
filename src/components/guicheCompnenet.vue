<template>
    <section>
       <div class="contentInOnlineUser">
            <div v-for="(user, index) in usersStatus" :key="index"  class="userContainer">
                    <div class="userInfoStatus">
                        <h1>{{ user.guiche }}</h1>
                        <h2>{{ user.atendente }}</h2>
             </div>
               

                <div class="currentCall" v-if="user.calls && user.calls.length">
                    <div v-for="(call, index) in user.calls" :key="index">
                        <h4>Chamando</h4>
                        <h3>Senha</h3>
                        <h2>{{ call.senha }}</h2>
                        <h3>Guiche</h3>
                        <h1>{{ call.guiche  }}</h1>
                       
                    </div>

                    <div class="timerRun" :style="{width: user.componentTimer + 'px', background: user.viewInfo }"></div>
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
            usersStatus: [],
            interval: null,
            intervalIds: {}
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
                if (data.status === 'sucess' && data.type === 'Call') {
                    const userIndex = this.usersStatus.findIndex(user => user.guiche === data.data.guiche);
                    if (userIndex !== -1) {
                        if (!this.usersStatus[userIndex].calls) {
                            this.usersStatus[userIndex].calls = [];
                        }
                        this.usersStatus[userIndex].calls.push(data.data);
                        this.resetTimer(userIndex);
                    }
                }

                if (data.status === 'success' && data.type === 'Delete') {
                    const deleteSenha = data.data;
                    this.usersStatus.forEach(user => {
                        if (user.calls) {
                            user.calls = user.calls.filter(call => call._id !== deleteSenha._id);
                        }
                    });
                }
            };
            ws.onerror = (error) => {
                console.error('WebSocket error:', error);
            };
            ws.onclose = () => {
                console.log('WebSocket closed');
            };
        },

        async getOnlineUsers() {
            try {
                const response = await axios.get('http://localhost:8080/getOnlineUsers');
                const online = response.data.data;
                this.usersStatus = online.map(user => ({
                    ...user,
                    componentTimer: 430,
                    viewInfo: '',
                    calls: []
                }));
                this.usersStatus.forEach((_, index) => this.startTimer(index));
            } catch (error) {
                console.error('error', error);
            }
        },

        async geTCall() {
            try {
                const response = await axios.get('http://localhost:8080/currentCalling');
                const calls = response.data.data;

                calls.forEach(call => {
                    const userIndex = this.usersStatus.findIndex(user => user.guiche === call.guiche);
                    if (userIndex !== -1) {
                        if (!this.usersStatus[userIndex].calls) {
                            this.usersStatus[userIndex].calls = [];
                        }
                        this.usersStatus[userIndex].calls.push(call);
                        this.resetTimer(userIndex);
                    }
                });
            } catch (error) {
                console.log(error);
            }
        },

        manageComponentTimer() {
            this.interval = setInterval(() => {
                this.usersStatus.forEach(user => {
                    if (user.componentTimer > 0) {
                        user.componentTimer --;
                        if (user.componentTimer <= 230) {
                            user.viewInfo = 'Orange';
                        }
                        if (user.componentTimer <= 100) {
                            user.viewInfo = 'red';
                        }
                    }else{
                        clearInterval(this.intervalIds[userIndex]);
                    }
                });
            }, 140);
        },

        resetTimer(userIndex) {
            clearInterval(this.intervalIds[userIndex]);
            this.usersStatus[userIndex].componentTimer = 430;
            this.usersStatus[userIndex].viewInfo = '';
            this.startTimer(userIndex);
        },


        beforeDestroy() {
            Object.values(this.intervalIds).forEach(clearInterval);
        },
    },

    mounted() {
        this.connectWebSocket();
        this.getOnlineUsers();
        this.geTCall();
        this.manageComponentTimer();
    }
}
</script>
