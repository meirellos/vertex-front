<template>
    <div>
        <div class="chat-widget">
            <div class="button-ivone" @click="showChat = !showChat">Converse com a IA</div>
            <div class="chat-modal" v-if="showChat">
                <div class="chat">
                    <div class="chat-title">
                        <span>Chat com Ivone - i9Educação</span>
                        <button @click="showChat = false">X</button>
                    </div>
                    <div class="chat-content">
                        <div class="chat-output">
                            <ul>
                                <li v-for="(message, index) in chatMessages" :key="index" class="mensagens">
                                    <div class="msg-user" v-if="index % 2 === 0">
                                        <div class="contato"><img src="@/assets/images/user.png" width="40px"
                                                height="auto"><strong>Você:</strong></div>
                                        <p>{{ message }}</p>
                                    </div>
                                    <div v-else class="msg-ivone">
                                        <div class="contato"> <img src="@/assets/images/ivone.svg" width="40px"
                                                height="auto"><strong>Ivone:</strong></div>
                                        <p v-html="marked(message)"></p>
                                    </div>
                                </li>
                            </ul>
                            <div v-if="isLoading" class="mensagens">
                                <div class="msg-ivone">
                                    <div class="contato">
                                        <img src="@/assets/images/ivone.svg" width="40px" height="auto">
                                        <strong>Ivone:</strong>
                                    </div>
                                    <div>A IA está pensando...</div>
                                </div>
                            </div>
                        </div>

                    </div>
                    <div class="chat-form">
                        <form @submit.prevent="sendMessage" class="formSendMessage">
                            <div class="chat-inputs">
                                <input type="text" v-model="message" placeholder="Digite sua mensagem..."
                                    @keyup.enter="sendMessage" :disabled="waitingResponse" />
                                <button type="submit" class="input-enviar" :disabled="waitingResponse">Enviar</button>
                            </div>
                            <input type="file" @change="handleFileUpload" ref="fileInput"
                                :disabled="waitingResponse"><br>
                            <button @click="clearHistory" class="clearHistory" :disabled="waitingResponse">Limpar
                                Histórico</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>


<style scoped>
.chat-widget {
    display: flex;
    justify-content: center;
    align-items: center;

}

.button-ivone {
    background: #2f71ff;
    padding: 20px;
    border-radius: 20px;
    color: #fff;
    cursor: pointer;
    font-size: 17px;
    font-weight: bold;
}

.button-ivone:hover {
    background: #4781fe;
}

.chat-modal {
    display: flex;
    align-items: center;
    justify-content: center;
    position: fixed;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.4);
    padding: 10px;
}

.chat {
    background: #dedede;
    border-radius: 10px;
    width: 70%;
    height: auto;
}

.chat-title {
    background: var(--standard-color-green-strong);
    border-radius: 10px 10px 0 0;
    color: var(--standard-color-black);
    font-weight: bold;
    padding: 20px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 22px;
}

.chat-title button {
    background: none;
    border: none;
    color: var(--standard-color-black);
    font-size: 20px;
    cursor: pointer;
}

.chat-content {
    background: #ffffff;
    padding: 10px;
    height: 500px;
    overflow: auto;
}

.chat-output ul {
    list-style: none;
    padding-inline-start: 0;
}

.chat-output ul li {
    display: flex;
    align-items: center;
    font-size: 17px;
    padding: 10px;
}

.msg-user {
    width: 100%;
    padding: 10px;
}

.contato {
    display: flex;
    align-items: center;
}

.contato strong {
    margin-left: 5px;
}

.msg-ivone {
    background: #f0f0f0;
    width: 100%;
    padding: 10px;
}

.formSendMessage {
    text-align: center;
}

.chat-form {
    background: #fff;
    box-shadow: rgba(0, 0, 0, 0.35) 0px 5px 15px;
    border-radius: 0 0 10px 10px;
    padding: 15px;
}

.chat-form input[type="text"] {
    background: #f0f0f0;
    border: none;
    border-radius: 10px;
    padding: 10px;
    margin: 10px;
    color: #929292;
    width: 300px;
}

.input-enviar {
    background: var(--standard-color-green-strong);
    padding: 10px;
    color: var(--standard-color-black);
    border: none;
    font-size: 17px;
    font-weight: 700;
    border-radius: 10px;
    cursor: pointer;
    padding-inline: 35px;
}

.input-enviar:hover {
    background: var(--standard-color-green);
}

@media(max-width:768px) {
    .chat-modal {
        width: auto;
    }

    .chat {
        width: 100%;
        height: auto;
    }
}
</style>

<script>
import axios from 'axios';
import { ref } from 'vue';
import { marked } from 'marked';

export default {
    mounted() {
        this.chatOutput = this.$el.querySelector('.chat-output ul');

        //Carregar as mensagens do localStorage (se tiver)
        const savedMessages = localStorage.getItem('chatMessages');
        if (savedMessages) {
            this.chatMessages = JSON.parse(savedMessages);
        }
    },
    setup() {
        const showChat = ref(false); //Inicia com o chat oculto.
        return {
            showChat
        }
    },
    data() {
        return {
            message: '',
            chatMessages: [],
            apiUrl: 'https://vertex-ai-teste-back-tnbkeowjkq-uc.a.run.app/api/generate',
            selectedFile: null,
            isLoading: false,
            waitingResponse: false
        };
    },
    methods: {
        handleFileUpload() {
            this.selectedFile = this.$refs.fileInput.files[0];
        },
        sendMessage() {
            if (this.message || this.selectedFile) {
                let messageData = { message: this.message };
                this.waitingResponse = true;
                if (this.selectedFile) {
                    this.getBase64(this.selectedFile)
                        .then(base64 => {
                            messageData.file = {
                                mimeType: this.selectedFile.type,
                                data: base64
                            };
                            this.sendMessageToNode(messageData);
                        })
                        .catch(error => {
                            console.error("Erro ao converter arquivo para Base64:", error);
                        });
                } else {
                    this.chatMessages.push(this.message);
                    this.sendMessageToNode(messageData);
                }
            }
        },
        sendMessageToNode(messageData) {
            this.isLoading = true; //Mostra a mensagem de aguarde a IA responder.
            this.waitingResponse = true; // Desabilita envio enquanto espera

            const delay = 5000; //Delay de mensagens (5 Segundos).
            // Envie a mensagem para o Node.js usando Axios

            setTimeout(() => {
                axios.post(this.apiUrl, messageData)
                    .then(response => {
                        // Receba a resposta da IA
                        const aiResponse = response.data.response;
                        const formattedResponse = aiResponse.replace(/<[^>]+>/g, '');

                        // Adicione a mensagem do usuário e a resposta da IA ao chat
                        //this.chatMessages.push(this.message);
                        this.chatMessages.push(formattedResponse);

                        //Salvando no localStorage
                        localStorage.setItem('chatMessages', JSON.stringify(this.chatMessages));

                        // Limpe o campo de entrada e atualize a interface
                        this.message = '';
                        //this.chatOutput.scrollTop = this.chatOutput.scrollHeight;
                    })
                    .catch(error => {
                        console.error('Erro ao enviar mensagem:', error);
                    })
                    .finally(() => {
                        this.message = '';
                        this.selectedFile = null;
                        this.$refs.fileInput.value = null;
                        this.isLoading = false; // Esconde a mensagem de espera
                        this.waitingResponse = false;
                    })

                // Atualize a rolagem da área de chat
                //this.chatOutput.scrollTop = this.chatOutput.scrollHeight;

            }, delay);
        },
        getBase64(file) {
            return new Promise((resolve, reject) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = () => resolve(reader.result.split(',')[1]); // Extraia apenas o base64
                reader.onerror = error => reject(error);
            });
        },
        clearHistory() {
            localStorage.removeItem('chatMessages');

            // Limpe o array de mensagens
            this.chatMessages = [];

            // Role a área de chat para o topo
            //this.chatOutput.scrollTop = 0;
        },
        marked(markdownString) {
            return marked(markdownString);
        }
    }
};
</script>