<template>
  <q-layout view="lHh Lpr lFf" class="fullscreen">
    <q-header bordered class="bg-grey-3 text-primary">
      <q-toolbar>
        <q-toolbar-title class="text-center">
     CONTADOR DEL REFRIGERADOR
          <q-avatar>
            <img src="https://cdn-icons-png.flaticon.com/512/642/642000.png">
            <h1 style="color:white">Contador de Refrigerador</h1>
          </q-avatar>
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-footer bordered class="bg-grey-3 text-primary">
      <q-tabs no-caps active-color="primary" indicator-color="transparent" class="text-grey-8" v-model="tab">
        <q-tab name="nevera" label="Nevera" icon="kitchen" />
        <q-tab name="estadisticas" label="Estadísticas" icon="bar_chart" />
      </q-tabs>
    </q-footer>

    <q-page-container>
      <q-page class="q-pa-md">
        <!-- Tab Nevera -->
        <div v-if="tab === 'nevera'">
          <q-card class="q-mb-md">
            <q-card-section class="text-center">
              <img :src="isOpen ? '/abierto.png' : '/cerrado.png'" :alt="isOpen ? 'Nevera Abierta' : 'Nevera Cerrada'" style="max-width: 300px; height: 300px;">
              <div class="text-h6 q-mt-md">Total de Aperturas: {{ totalOpens }}</div>
              <div class="q-mt-md">
                <q-btn color="primary" size="lg" label="Abrir Refrigerador" @click="incrementCounter" class="q-mr-md" />
                <q-btn color="negative" size="lg" label="Reiniciar Contador" @click="resetCounter" />
              </div>
              <div v-if="judgmentMessage" class="text-body2 q-mt-sm text-grey-7">{{ judgmentMessage }}</div>
            </q-card-section>
          </q-card>
        </div>

        <!-- Tab Estadísticas -->
        <div v-if="tab === 'estadisticas'">
          <q-card class="q-mb-md">
            <q-card-section>
              <div class="text-h6">Estadísticas Detalladas</div>
              <q-separator class="q-my-md" />
              <div class="row q-gutter-md">
                <div class="col-6">
                  <q-card flat bordered class="text-center">
                    <q-card-section>
                      <div class="text-h6 text-primary">{{ todayOpens }}</div>
                      <div class="text-caption">Total Hoy</div>
                    </q-card-section>
                  </q-card>
                </div>
                <div class="col-6">
                  <q-card flat bordered class="text-center">
                    <q-card-section>
                      <div class="text-h6 text-primary">{{ weeklyAverage }}</div>
                      <div class="text-caption">Promedio Semanal</div>
                    </q-card-section>
                  </q-card>
                </div>
              </div>
            </q-card-section>
          </q-card>

          <q-card class="q-mb-md">
            <q-card-section>
              <div class="text-h6">Aperturas por Hora (Hoy)</div>
              <apexchart type="bar" :options="hourlyChartOptions" :series="hourlyChartSeries" height="300" />
            </q-card-section>
          </q-card>

          <q-card>
            <q-card-section>
              <div class="text-h6">Patrones Semanales</div>
              <apexchart type="bar" :options="weeklyChartOptions" :series="weeklyChartSeries" height="300" />
            </q-card-section>
          </q-card>
        </div>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import ApexChart from 'vue3-apexcharts'

const tab = ref('nevera') // Tab inicial

// Estado reactivo
const totalOpens = ref(0)
const openTimes = ref([])
const isOpen = ref(false)

// Cargar datos de localStorage al montar
onMounted(() => {
  const stored = localStorage.getItem('refrigeratorOpens')
  if (stored) {
    const data = JSON.parse(stored)
    totalOpens.value = data.total
    openTimes.value = data.times.map(t => new Date(t))
  }
})

// Función para incrementar contador
const incrementCounter = () => {
  isOpen.value = true
  setTimeout(() => {
    isOpen.value = false
  }, 500)
  totalOpens.value++
  openTimes.value.push(new Date())
  localStorage.setItem('refrigeratorOpens', JSON.stringify({
    total: totalOpens.value,
    times: openTimes.value
  }))
  updateJudgment()
}

// Función para reiniciar contador
const resetCounter = () => {
  totalOpens.value = 0
  openTimes.value = []
  localStorage.removeItem('refrigeratorOpens')
  judgmentMessage.value = ''
}

// Estadísticas
const todayOpens = computed(() => {
  const today = new Date().toDateString()
  return openTimes.value.filter(t => t.toDateString() === today).length
})

const weeklyAverage = computed(() => {
  const now = new Date()
  const weekAgo = new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000)
  const weekOpens = openTimes.value.filter(t => t >= weekAgo).length
  return (weekOpens / 7).toFixed(1)
})

// Estadísticas por hora (hoy) - Siempre muestra 0-23
const hourlyStats = computed(() => {
  const today = new Date().toDateString()
  const stats = {}
  for (let h = 0; h < 24; h++) {
    stats[`${h}`] = openTimes.value.filter(t => t.toDateString() === today && t.getHours() === h).length
  }
  return stats
})

// Estadísticas semanales - Siempre muestra días
const weeklyStats = computed(() => {
  const days = ['Domingo', 'Lunes', 'Martes', 'Miércoles', 'Jueves', 'Viernes', 'Sábado']
  const stats = {}
  days.forEach((day, i) => {
    stats[day] = openTimes.value.filter(t => t.getDay() === i).length
  })
  return stats
})

// Mensaje de juicio
const judgmentMessage = ref('')
const judgmentMessages = [
  'Es la cuarta vez en 15 minutos. La comida no se multiplicó mágicamente.',
  '¿Otra vez? ¿Ya sembraste la comida para que creciera?',
  'Yo no soy un slot machine. Entre más me abres, menos probabilidades hay de que ganes algo.',
  'Bienvenido al museo de las sobras. La exposición es la misma de hace 10 minutos.',
  'Cada vez que abres mi puerta, un cubito de hielo pierde la fe en la humanidad.',
  'No, el pollo no se cocinó solo. Sigue crudo y decepcionado.',
  'Este no es un reality show. No hay un drama nuevo entre el yogur y la leche.',
  '¿Qué esperabas? ¿Que el brócoli te saludara? Hola, ¡soy aún más verde!',
  '¡Por favor! Estaba en plena criogenización de mis sueños. Ahora se descongeló mi esperanza de ser una lavadora.',
  '¡Qué hambre constante!',
  '¿Estás seguro de que necesitas abrirlo?',
  '¡Wow, eres un experto en refrigeradores!',
  '¡El refrigerador te extraña!',
  '¡Otra apertura más!'
]

const updateJudgment = () => {
  const randomIndex = Math.floor(Math.random() * judgmentMessages.length)
  judgmentMessage.value = judgmentMessages[randomIndex]
}

// Gráficos ApexCharts
const hourlyChartOptions = computed(() => ({
  chart: {
    type: 'bar',
    height: 300
  },
  xaxis: {
    categories: Array.from({ length: 24 }, (_, i) => `${i}:00`)
  },
  yaxis: {
    title: {
      text: 'Aperturas'
    }
  },
  colors: ['#1976d2']
}))

const hourlyChartSeries = computed(() => [{
  name: 'Aperturas',
  data: Object.values(hourlyStats.value)
}])

const weeklyChartOptions = computed(() => ({
  chart: {
    type: 'bar',
    height: 300
  },
  xaxis: {
    categories: ['Domingo', 'Lunes', 'Martes', 'Miércoles', 'Jueves', 'Viernes', 'Sábado']
  },
  yaxis: {
    title: {
      text: 'Aperturas'
    }
  },
  colors: ['#42a5f5']
}))

const weeklyChartSeries = computed(() => [{
  name: 'Aperturas',
  data: Object.values(weeklyStats.value)
}])
</script>

<style scoped>
.fullscreen {
  height: 100vh;
}
</style>
