<template>
  <div class="flex flex-col min-h-screen bg-gray-100">
    
    <!-- <header class="bg-white shadow-md">
      <div class="mx-2 p-3 flex justify-between items-center">
        <h1 class="text-2xl font-semibold text-gray-800">Dashboard</h1>
        <div class="flex space-x-4">
          <button @click="downloadExcel" class="bg-green-800 text-white text-sm font-semibold  px-2 py-2 rounded-lg hover:bg-green-600">
            Download Excel
          </button>
          <button @click="downloadPDF" class="bg-red-800 text-white text-sm px-2 py-2 rounded-lg hover:bg-red-600">
            Download PDF
          </button>
        </div>
      </div>
    </header> -->

    <!-- Main Content -->
    <main class="flex-1 p-7 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-2 gap-6">
      <!-- Chart Container -->
      <div class="bg-white shadow-lg rounded-lg p-6">
        <div class="flex justify-between items-center mb-4">
          <h2 class="text-xl font-semibold">Meat Distribution Chart</h2>
          <button
            @click="toggleChartType"
            class="bg-blue-500 text-white text-sm px-1 py-2 rounded-lg hover:bg-blue-600 transition"
          >
            Toggle {{ isBarChart ? 'Line' : 'Bar' }} Chart
          </button>
        </div>
        <!-- Chart -->
        <component :is="chartComponent" v-if="chartData.datasets[0].data.length" :data="chartData" :options="chartOptions" />
      </div>

      <!-- Summary Container -->
      <div class="flex flex-col gap-4">
        <div class="bg-blue-500 text-white p-4 rounded-lg shadow-md">
          <h2 class="text-xl font-semibold mb-2">Average Sales Per Month</h2>
          <ul class="list-none p-0 m-0 space-y-2">
            <li class="flex justify-between items-center">
              <span>Duck</span>
              <span>{{ averageSales.duck }} units</span>
            </li>
            <li class="flex justify-between items-center">
              <span>Chicken</span>
              <span>{{ averageSales.chicken }} units</span>
            </li>
            <li class="flex justify-between items-center">
              <span>Lamb</span>
              <span>{{ averageSales.lamb }} units</span>
            </li>
            <li class="flex justify-between items-center">
              <span>Beef</span>
              <span>{{ averageSales.beef }} units</span>
            </li>
            <li class="flex justify-between items-center">
              <span>Pork</span>
              <span>{{ averageSales.pork }} units</span>
            </li>
          </ul>
        </div>
        <div class="bg-gray-200 p-4 rounded-lg shadow-md">
          <h2 class="text-xl font-semibold mb-2">Total Revenue & Sales</h2>
          <ul class="list-none p-0 m-0 space-y-2">
            <li class="flex justify-between items-center">
              <span>Total Sales</span>
              <span>{{ totalSales }} units</span>
            </li>
            <li class="flex justify-between items-center">
              <span>Total Revenue</span>
              <span>${{ totalRevenue.toFixed(2) }}</span>
            </li>
          </ul>
        </div>
      </div>
    </main>

    <!-- Footer -->
    <footer class="bg-white shadow-md mt-4 py-4">
      <div class="container mx-auto text-center text-gray-600">
        <p>&copy; 2024 Report App.</p>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { Bar, Line, Doughnut } from 'vue-chartjs';
import { Chart as ChartJS, CategoryScale, LinearScale, BarElement, LineElement, PointElement, ArcElement, Title, Tooltip, Legend } from 'chart.js';
import axios from 'axios';
import * as XLSX from 'xlsx';
import jsPDF from 'jspdf';
import html2canvas from 'html2canvas';

// Register the components required for the charts
ChartJS.register(CategoryScale, LinearScale, BarElement, LineElement, PointElement, ArcElement, Title, Tooltip, Legend);
definePageMeta({
  middleware: ['auth'],
  layout: 'default'
})
// Define refs for chart data, options, sales data, average sales, total revenue, and total sales
const chartData = ref({
  labels: [],
  datasets: [{
    label: 'Meat Distribution',
    data: [],
    backgroundColor: [
      'rgba(75, 192, 192, 0.2)',  // Duck
      'rgba(255, 99, 132, 0.2)',   // Chicken
      'rgba(153, 102, 255, 0.2)',  // Lamb
      'rgba(255, 159, 64, 0.2)',   // Beef
      'rgba(255, 205, 86, 0.2)',   // Pork
    ],
    borderColor: [
      'rgba(75, 192, 192, 1)',    // Duck
      'rgba(255, 99, 132, 1)',    // Chicken
      'rgba(153, 102, 255, 1)',   // Lamb
      'rgba(255, 159, 64, 1)',    // Beef
      'rgba(255, 205, 86, 1)',    // Pork
    ],
    borderWidth: 1,
  }]
});

const chartOptions = {
  responsive: true,
  plugins: {
    legend: {
      position: 'top',
    },
    tooltip: {
      callbacks: {
        label: function(context) {
          return `${context.label}: ${context.raw}`;
        }
      }
    },
    datalabels: {
      color: '#fff',
      display: true,
      formatter: (value) => `${value}`,
      font: {
        weight: 'bold',
      },
      anchor: 'center',
      align: 'center',
    },
  },
};

const averageSales = ref({
  duck: 0,
  chicken: 0,
  lamb: 0,
  beef: 0,
  pork: 0,
});
const totalRevenue = ref(0);
const totalSales = ref(0);
const isBarChart = ref(true);

// Determine which chart component to use
const chartComponent = computed(() => (isBarChart.value ? Bar : Line));

// Fetch data from the API and process it
const fetchData = async () => {
  try {
    const response = await axios.get('https://run.mocky.io/v3/f3a4fb16-52a0-46db-b1f7-1ce65259998d');
    console.log('API Response:', response.data);  // Log the entire API response for debugging

    const sales_data = response.data.sales_data;
    if (!sales_data || !Array.isArray(sales_data)) {
      throw new Error('Invalid data format from API');
    }

    // Process the data to sum up values
    const duckData = sales_data.reduce((acc, item) => acc + (item.duck || 0), 0);
    const chickenData = sales_data.reduce((acc, item) => acc + (item.chicken || 0), 0);
    const lambData = sales_data.reduce((acc, item) => acc + (item.lamb || 0), 0);
    const beefData = sales_data.reduce((acc, item) => acc + (item.beef || 0), 0);
    const porkData = sales_data.reduce((acc, item) => acc + (item.pork || 0), 0);

    // Update chart data
    chartData.value.labels = ['Duck', 'Chicken', 'Lamb', 'Beef', 'Pork'];
    chartData.value.datasets[0].data = [duckData, chickenData, lambData, beefData, porkData];

    // Calculate average sales per month
    const months = new Set(sales_data.map(item => new Date(item.date).toLocaleString('default', { month: 'short', year: 'numeric' })));
    const monthCount = months.size;

    averageSales.value.duck = Math.round(duckData / monthCount / 2) * 2;
    averageSales.value.chicken = Math.round(chickenData / monthCount / 2) * 2;
    averageSales.value.lamb = Math.round(lambData / monthCount / 2) * 2;
    averageSales.value.beef = Math.round(beefData / monthCount / 2) * 2;
    averageSales.value.pork = Math.round(porkData / monthCount / 2) * 2;

    // Calculate total revenue and total sales
    totalRevenue.value = sales_data.reduce((acc, item) => acc + (item.duck * 10 + item.chicken * 15 + item.lamb * 20 + item.beef * 25 + item.pork * 12), 0); // Sample pricing
    totalSales.value = duckData + chickenData + lambData + beefData + porkData;

    console.log('Processed Chart Data:', chartData.value);  // Log the chart data for debugging
    console.log('Average Sales Per Month:', averageSales.value);  // Log the average sales for debugging
    console.log('Total Revenue:', totalRevenue.value);  // Log the total revenue for debugging
    console.log('Total Sales:', totalSales.value);  // Log the total sales for debugging

  } catch (error) {
    console.error('Error fetching data:', error);
  }
};

// Fetch data on component mount
onMounted(() => {
  fetchData();
});

// Function to download data as Excel file
const downloadExcel = () => {
  const data = [
    ['Type of Meat', 'Average Sales Per Month'],
    ['Duck', averageSales.value.duck],
    ['Chicken', averageSales.value.chicken],
    ['Lamb', averageSales.value.lamb],
    ['Beef', averageSales.value.beef],
    ['Pork', averageSales.value.pork],
    ['Total Sales', totalSales.value],
    ['Total Revenue', totalRevenue.value.toFixed(2)]
  ];

  const ws = XLSX.utils.aoa_to_sheet(data);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, 'Meat Sales Data');
  XLSX.writeFile(wb, 'meat_sales_data.xlsx');
};

// Function to download data as PDF file
const downloadPDF = () => {
  html2canvas(document.querySelector('main')).then(canvas => {
    const imgData = canvas.toDataURL('image/png');
    const pdf = new jsPDF();
    pdf.addImage(imgData, 'PNG', 10, 10, 190, 0); // Add the screenshot to the PDF
    pdf.save('meat_sales_data.pdf'); // Save the PDF
  });
};
// Function to toggle between bar and line charts
const toggleChartType = () => {
  isBarChart.value = !isBarChart.value;
};
</script>

<style scoped>
/* Add your styles here */
</style>


<!-- <script setup lang="ts">
definePageMeta({
  middleware: ['auth']
})
</script> -->
