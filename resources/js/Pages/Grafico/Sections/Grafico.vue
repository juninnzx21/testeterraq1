<template>
    <!-- Bar Chart Component -->
    <div>
      <Bar
        id="my-chart-id"
        :options="chartOptions"
        :data="chartData"
      />
    </div>
  </template>
  
  <script setup>
    // Importando módulos necessários do Vue e Chart.js
    import { reactive, defineProps, onMounted } from 'vue';
    import { Bar } from 'vue-chartjs';
    import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js';
  
    // Registrando os componentes necessários do Chart.js
    ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale);
  
    // Definindo propriedades recebidas
    const { dados } = defineProps(['dados']);
  
    // Criando dados reativos para o gráfico
    const chartData = reactive({
      labels: dados.labels,
      datasets: dados.datasets
    });
  
    // Configurações reativas do gráfico
    const chartOptions = reactive({
      responsive: true
    });
  </script>
  
  <script>
    // Importando módulos adicionais do Vue
    import { defineComponent } from 'vue';
  
    // Definindo e exportando o componente BarChart
    export default defineComponent({
      name: 'BarChart',
  
      // Registrando o componente Bar
      components: { Bar },
  
      // Configuração do componente
      setup() {
        // Retornando dados e opções para o template
        return {
          chartData,
          chartOptions,
        };
      }
    });
  </script>
  