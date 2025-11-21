<template>

<div class="padre">

    <div class="q-pa-md">
    <q-scroll-area style="height: calc(100vh - 160px); max-width: 2000px;">
      <div class="q-pa-md row justify-center">
        <div style="width: 100%; max-width: 400px">
          <div v-for="(message, index) in messages" :key="index">
            <q-chat-message
              :name="message.name"
              :avatar="message.avatar"
              :text="[message.text]"
              :stamp="message.stamp"
              :sent="message.sent"
              :text-color="message.textColor"
              :bg-color="green"
            />
          </div>
          <div class="q-mt-md">
            <q-input
              v-model="userInput"
              label="Preguntale a Cháchara"
              outlined
              @keyup.enter="sendMessage"
            />
            <q-btn
              label="Enviar"
              color="primary"
              @click="sendMessage"
              class="q-mt-sm"
            />
          </div>
        </div>
      </div>
    </q-scroll-area>
  </div>

</div>
<q-dialog v-model="showIntroDialog" persistent maximized>
  <q-card class="bg-green">  <!-- Fondo verde -->
    <div class="relative">  <!-- Contenedor relativo -->
      <q-img src="/chachara.png" style="width: 1920px; height: 945px;" />
      
      <!-- Texto debajo del botón (posicionado absolutamente cerca del fondo) -->
      <q-card-section class="absolute text-center" style="bottom: 80px; left: 50%; transform: translateX(-50%); z-index: 20;" >
        <span style="color:darkgreen; font-size:25px; font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;">¡Bienvenido! listo para hablar con tu consejero de confianza Cháchara 😁👍, te ayudara en lo que sea tu solo pregunta 
          quieres empezar 😎?.</span>
      </q-card-section>
      <!-- Botón en la parte inferior de la imagen -->
      <q-card-actions class="absolute-bottom text-center">
        <q-btn
  flat label="Empecemos!"
  color="green-10"
  background-color="white"
  v-close-popup
  size="lg"
  class="q-pa-md text-weight-bold text-uppercase rounded-borders shadow-4"
  style="font-size: 30px; padding: 12px 24px;"
  font-family="SuperChiby"
/>

      </q-card-actions>
    </div>
  </q-card>
</q-dialog>

</template>

<script>
import { ref } from 'vue'

export default {
  name: 'App',
  setup() {
    const userInput = ref('')
    const messages = ref([
      {
        name: 'Cháchara',
        avatar: 'https://img.freepik.com/vector-premium/monstruo-dibujos-animados-expresion-facial-divertida-emocionada-agitando-manos-ilustracion-vectorial-aislada-diseno-blanco-halloween_6996-6410.jpg',
        text: 'ola! soy Cháchara, adelante, pregunta lo que sea! te dare los mejores consejos 😀👌😌',
        stamp: new Date().toLocaleTimeString(),
        sent: false,
        textColor: 'white',
        bgColor: 'primary'
      }
    ])
    const showIntroDialog = ref(true)  // Add this for the intro dialog

    const sendMessage = async () => {
      if (userInput.value.trim() === '') return

      // Agregar mensaje del usuario
      messages.value.push({
        name: 'Tú',
        avatar: 'https://static.vecteezy.com/system/resources/previews/013/042/571/non_2x/default-avatar-profile-icon-social-media-user-photo-in-flat-style-vector.jpg',
        text: userInput.value,
        stamp: new Date().toLocaleTimeString(),
        sent: true,
        bgColor: 'amber-7'
      })

      const question = userInput.value
      userInput.value = ''

      // Generar respuesta mala con Gemini (usando función global)
      const advice = await window.generateBadAdvice(question)

      // Agregar respuesta del asistente
      messages.value.push({
        name: 'Cháchara',
        avatar: 'https://img.freepik.com/vector-premium/monstruo-dibujos-animados-expresion-facial-divertida-emocionada-agitando-manos-ilustracion-vectorial-aislada-diseno-blanco-halloween_6996-6410.jpg',
        text: advice,
        stamp: new Date().toLocaleTimeString(),
        sent: false,
        textColor: 'white',
        bgColor: 'primary'
      })
    }

    return {
      userInput,
      messages,
      sendMessage,
      showIntroDialog  // Add this to return
    }
  }
}
</script>


<style scoped>

.padre {
  padding: 80px;
  border-radius: 2px;
  border-color: green;
}


.relative {
  position: relative;
}
.absolute {
  position: absolute;
  z-index: 10;
}

.styled-btn {
  background: linear-gradient(45deg, #4CAF50, #66BB6A);
  border-radius: 20px;
  transition: transform 0.2s;
}
.styled-btn:hover {
  transform: scale(1.05);
}
 
.custom-font {
  font-family: 'SuperChiby', sans-serif;
}

</style>



https://img.freepik.com/free-vector/vibrant-pattern-design_1409-9648.jpg?semt=ais_hybrid&w=740&q=80


https://www.shutterstock.com/image-vector/no-plastic-go-green-zero-600nw-1950569695.jpg