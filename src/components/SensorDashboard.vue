<template>
  <div class="webapp-layout">
    <!-- Zijbalk (Sidebar) -->
    <aside class="sidebar">
      <div class="brand">
        <span class="logo-icon">🌿</span>
        <h2>HydroFarm</h2>
      </div>
      <nav class="nav-menu">
        <a href="#" class="active">📊 Dashboard</a>
      </nav>
    </aside>

    <!-- Hoofdinhoud -->
    <main class="main-content">
      <!-- Header -->
      <header class="dashboard-header">
        <div>
          <h1 class="title">Vertical Farm Monitor</h1>
          <p class="subtitle">Real-time sensor data en analyse</p>
        </div>
        <div class="status-badge" :class="{ 'is-live': !loading }">
          <span class="pulse"></span>
          {{ loading ? 'Sensoren uitlezen...' : 'Systeem Online' }}
        </div>
      </header>

      <!-- Sensor Grid -->
      <div class="sensor-grid">
        <div class="card temp-card">
          <div class="card-icon temp-icon">🌡️</div>
          <div class="card-content">
            <span class="label">Temperatuur</span>
            <div class="value-container">
              <span class="value">{{ sensorData.temperature.toFixed(1) }}</span>
              <span class="unit">°C</span>
            </div>
          </div>
        </div>

        <div class="card ph-card">
          <div class="card-icon ph-icon">💧</div>
          <div class="card-content">
            <span class="label">pH Waarde</span>
            <div class="value-container">
              <span class="value">{{ sensorData.ph.toFixed(2) }}</span>
              <span class="unit">pH</span>
            </div>
          </div>
        </div>

        <div class="card tds-card">
          <div class="card-icon tds-icon">🌱</div>
          <div class="card-content">
            <span class="label">Voeding (TDS)</span>
            <div class="value-container">
              <span class="value">{{ Math.round(sensorData.tds) }}</span>
              <span class="unit">ppm</span>
            </div>
          </div>
        </div>

        <div class="card water-card">
          <div class="card-icon water-icon">📏</div>
          <div class="card-content">
            <span class="label">Waterniveau</span>
            <div class="value-container">
              <span class="value">{{ Math.round(sensorData.distance) }}</span>
              <span class="unit">mm</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Uitgebreide sectie: Grafiek & Logs -->
      <div class="extended-grid">
        <!-- Grafiek -->
        <div class="chart-section">
          <h2 class="section-title">Temperatuur & pH Verloop</h2>
          <div class="chart-container">
            <canvas ref="chartCanvas"></canvas>
          </div>
        </div>

        <!-- Logboek -->
        <div class="log-section">
          <h2 class="section-title">Systeem Logboek</h2>
          <div class="log-list">
            <div v-for="(log, index) in logs" :key="index" class="log-item" :class="log.type">
              <span class="log-time">{{ log.time }}</span>
              <span class="log-msg">{{ log.message }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Footer -->
      <footer class="footer-info">
        Laatste update: {{ lastUpdated }} | Device ID: <strong>{{ deviceId }}</strong>
      </footer>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import Chart from 'chart.js/auto';

// --- Status & Config ---
const deviceId = ref('devicelars');
const loading = ref(false);
const lastUpdated = ref('Zojuist');
let pollingInterval: number | undefined;
let chartInstance: any = null;
const chartCanvas = ref<HTMLCanvasElement | null>(null);

// --- State (Startwaardes) ---
const sensorData = ref({
  distance: 120,
  ph: 6.0,
  tds: 650,
  temperature: 21.5
});

const logs = ref([
  { time: new Date().toLocaleTimeString('nl-NL'), message: 'App opgestart. Sensoren kalibreren...', type: 'info' }
]);

const history = {
  labels: [] as string[],
  temp: [] as number[],
  ph: [] as number[]
};

// --- Methoden ---
const addLog = (message: string, type = 'info') => {
  logs.value.unshift({
    time: new Date().toLocaleTimeString('nl-NL'),
    message,
    type
  });
  if (logs.value.length > 20) logs.value.pop();
};

const updateChart = () => {
  if (!chartInstance) return;
  const now = new Date().toLocaleTimeString('nl-NL', { hour: '2-digit', minute: '2-digit', second: '2-digit' });

  history.labels.push(now);
  history.temp.push(sensorData.value.temperature);
  history.ph.push(sensorData.value.ph);

  if (history.labels.length > 20) {
    history.labels.shift();
    history.temp.shift();
    history.ph.shift();
  }
  chartInstance.update();
};

const fetchSensorData = async () => {
  // === ECHTE DATA (NODE-RED) ===
  // Haal de comments hieronder weg als Node-RED werkt. 
  // Zet dan de DUMMY DATA code hieronder in comments!

  /*
  try {
    loading.value = true;
    const response = await axios.get(`http://JOUW-NODE-RED-IP:1880/api/sensors/${deviceId.value}`);
    sensorData.value = response.data;
    lastUpdated.value = new Date().toLocaleTimeString('nl-NL');
    updateChart();
  } catch (error) {
    addLog('API Verbindingsfout met Node-RED', 'warn');
  } finally {
    loading.value = false;
  }
  */

  // === DUMMY DATA SIMULATIE ===
  loading.value = true;
  setTimeout(() => {
    sensorData.value.temperature += (Math.random() - 0.5) * 0.4;
    sensorData.value.ph += (Math.random() - 0.5) * 0.1;
    sensorData.value.tds += (Math.random() - 0.5) * 5;
    sensorData.value.distance += (Math.random() - 0.5) * 2;

    lastUpdated.value = new Date().toLocaleTimeString('nl-NL');
    loading.value = false;
    updateChart();

    if (sensorData.value.ph > 6.5) {
      addLog('Waarschuwing: pH niveau stijgt boven 6.5', 'warn');
    }
  }, 200);
};

const initChart = () => {
  if (chartCanvas.value) {
    chartInstance = new Chart(chartCanvas.value, {
      type: 'line',
      data: {
        labels: history.labels,
        datasets: [
          {
            label: 'Temperatuur (°C)',
            data: history.temp,
            borderColor: '#ef4444',
            backgroundColor: 'rgba(239, 68, 68, 0.1)',
            yAxisID: 'y',
            tension: 0.4,
            fill: true
          },
          {
            label: 'pH Waarde',
            data: history.ph,
            borderColor: '#8b5cf6',
            backgroundColor: 'rgba(139, 92, 246, 0.1)',
            yAxisID: 'y1',
            tension: 0.4,
            fill: true
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        interaction: { mode: 'index', intersect: false },
        scales: {
          y: { type: 'linear', display: true, position: 'left', min: 18, max: 26 },
          y1: { type: 'linear', display: true, position: 'right', min: 4, max: 8, grid: { drawOnChartArea: false } }
        }
      }
    });
  }
};

onMounted(() => {
  initChart();
  fetchSensorData();
  pollingInterval = setInterval(fetchSensorData, 3000);
});

onUnmounted(() => {
  clearInterval(pollingInterval);
  if (chartInstance) chartInstance.destroy();
});
</script>

<!-- GLOBALE CSS (Voor het verwijderen van de witte balk en centrering) -->
<style>
body,
html {
  margin: 0;
  padding: 0;
  background-color: #f8fafc;
  overflow: hidden;
}

#app {
  max-width: none !important;
  padding: 0 !important;
  margin: 0 !important;
  display: block !important;
  width: 100vw;
  height: 100vh;
}
</style>

<!-- LOKALE CSS VOOR DIT COMPONENT -->
<style scoped>
.webapp-layout {
  display: flex;
  height: 100vh;
  width: 100vw;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  background: #f8fafc;
  overflow: hidden;
}

/* Sidebar Styling */
.sidebar {
  width: 260px;
  background-color: #0f172a;
  color: #ffffff;
  display: flex;
  flex-direction: column;
  padding: 24px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 40px;
}

.logo-icon {
  font-size: 2rem;
}

.brand h2 {
  margin: 0;
  font-size: 1.4rem;
  font-weight: 700;
  color: #f8fafc;
}

.nav-menu {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.nav-menu a {
  color: #94a3b8;
  text-decoration: none;
  padding: 12px 16px;
  border-radius: 10px;
  font-weight: 500;
  transition: all 0.2s ease;
}

.nav-menu a:hover {
  background-color: #1e293b;
  color: #ffffff;
}

.nav-menu a.active {
  background-color: #3b82f6;
  color: #ffffff;
}

/* Main Content Area */
.main-content {
  flex: 1;
  padding: 40px;
  overflow-y: auto;
}

.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 40px;
}

.title {
  margin: 0;
  font-size: 2rem;
  color: #0f172a;
  font-weight: 700;
}

.subtitle {
  margin: 8px 0 0 0;
  color: #64748b;
  font-size: 1rem;
}

.status-badge {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 20px;
  background-color: #f1f5f9;
  color: #64748b;
  border-radius: 50px;
  font-weight: 600;
  font-size: 0.9rem;
}

.status-badge.is-live {
  background-color: #dcfce7;
  color: #166534;
}

.pulse {
  width: 12px;
  height: 12px;
  background-color: #94a3b8;
  border-radius: 50%;
}

.is-live .pulse {
  background-color: #22c55e;
  box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.7);
  animation: pulse-animation 2s infinite;
}

@keyframes pulse-animation {
  0% {
    box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.7);
  }

  70% {
    box-shadow: 0 0 0 10px rgba(34, 197, 94, 0);
  }

  100% {
    box-shadow: 0 0 0 0 rgba(34, 197, 94, 0);
  }
}

.sensor-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 24px;
  margin-bottom: 30px;
}

.card {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  padding: 24px;
  display: flex;
  align-items: center;
  gap: 20px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
}

.card-icon {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 56px;
  height: 56px;
  border-radius: 16px;
  font-size: 1.6rem;
}

.temp-icon {
  background: #fee2e2;
}

.ph-icon {
  background: #f3e8ff;
}

.tds-icon {
  background: #fef9c3;
}

.water-icon {
  background: #e0f2fe;
}

.card-content {
  display: flex;
  flex-direction: column;
}

.label {
  font-size: 0.85rem;
  color: #64748b;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.value-container {
  display: flex;
  align-items: baseline;
  gap: 6px;
  margin-top: 4px;
}

.value {
  font-size: 2.2rem;
  font-weight: 800;
  color: #0f172a;
}

.unit {
  font-size: 1.1rem;
  color: #94a3b8;
  font-weight: 600;
}

/* Extended Section: Grafiek & Logs */
.extended-grid {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 24px;
  width: 100%;
}

.chart-section,
.log-section {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
  min-width: 0;
  /* Belangrijk: zorgt dat de grafiek kan krimpen */
}

.section-title {
  margin: 0 0 20px 0;
  font-size: 1.2rem;
  color: #0f172a;
  font-weight: 700;
}

.chart-container {
  position: relative;
  height: 350px;
  width: 100%;
}

.log-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  max-height: 350px;
  overflow-y: auto;
}

.log-item {
  display: flex;
  flex-direction: column;
  padding: 14px;
  border-radius: 8px;
  background: #f8fafc;
  border-left: 4px solid #cbd5e1;
  font-size: 0.9rem;
}

.log-item.info {
  border-left-color: #3b82f6;
}

.log-item.warn {
  border-left-color: #f59e0b;
  background: #fffbeb;
}

.log-time {
  color: #94a3b8;
  font-size: 0.8rem;
  margin-bottom: 4px;
}

.log-msg {
  color: #334155;
  font-weight: 500;
}

.footer-info {
  margin-top: 40px;
  padding-top: 20px;
  border-top: 1px solid #e2e8f0;
  text-align: right;
  font-size: 0.9rem;
  color: #94a3b8;
}

.footer-info strong {
  color: #64748b;
}

/* Responsiviteit */
@media (max-width: 1100px) {
  .extended-grid {
    grid-template-columns: 1fr;
  }

  .sidebar {
    width: 80px;
  }

  .sidebar h2,
  .nav-menu a {
    display: none;
  }

  .nav-menu a::after {
    content: "📋";
    display: block;
    font-size: 1.5rem;
    text-align: center;
  }

  .chart-container {
    height: 250px;
  }
}

@media (max-width: 600px) {
  .main-content {
    padding: 20px;
  }

  .chart-section,
  .log-section {
    padding: 16px;
  }

  .chart-container {
    height: 200px;
  }
}
</style>