<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read the documentation</p>
<script>
  import { onMount } from 'svelte';
  import Chart from 'chart.js/auto';

  let city = '';
  let weather = null;
  let forecast = [];
  let error = '';

  const API_KEY = '46903166240b9f3399bc603ba45f58fe'; // 🔁 Replace with your OpenWeatherMap API key

  let chartCanvas;
  let chartInstance;

  async function searchWeather() {
    error = '';
    weather = null;
    forecast = [];

    const currentUrl = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`;
const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${API_KEY}&units=metric`;


    try {
      const res1 = await fetch(currentUrl);
      if (!res1.ok) throw new Error('City not found');
      weather = await res1.json();

      const res2 = await fetch(forecastUrl);
      if (!res2.ok) throw new Error('Forecast not found');
      const forecastData = await res2.json();

      forecast = forecastData.list.filter((_, index) => index % 8 === 0); // every 24 hours
      console.log('Forecast:', forecast);
      updateChart();
    } catch (err) {
      error = err.message;
    }
  }

function updateChart() {
  if (!forecast.length) return;

  // Use requestAnimationFrame to make sure canvas is in DOM
  requestAnimationFrame(() => {
    const ctx = chartCanvas?.getContext('2d');
    if (!ctx) {
      console.error('Canvas context not available');
      return;
    }

    const labels = forecast.map(day => day.dt_txt.split(' ')[0]);
    const temps = forecast.map(day => day.main.temp);
    const humidity = forecast.map(day => day.main.humidity);

    if (chartInstance) {
      chartInstance.destroy();
    }

    chartInstance = new Chart(ctx, {
      type: 'line',
      data: {
        labels,
        datasets: [
          {
            label: 'Temperature (°C)',
            data: temps,
            borderColor: 'orange',
            fill: false
          },
          {
            label: 'Humidity (%)',
            data: humidity,
            borderColor: 'blue',
            fill: false
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false
      }
    });
  });
}
</script>

<main>
  <h1>Weather Tracker 🌦</h1>

  <form on:submit|preventDefault={searchWeather}>
    <input
      type="text"
      placeholder="Enter city name"
      bind:value={city}
      required
    />
    <button type="submit">Search</button>
  </form>

  {#if error}
    <p style="color: red;">{error}</p>
  {/if}

  {#if weather}
    <div>
      <h2>{weather.name}, {weather.sys.country}</h2>
      <p>🌡 Temperature: {weather.main.temp}°C</p>
      <p>💧 Humidity: {weather.main.humidity}%</p>
      <p>🌥 Condition: {weather.weather[0].description}</p>
    </div>
  {/if}

  {#if forecast.length}
    <h3>5-Day Forecast</h3>
    <ul>
      {#each forecast as day}
        <li>
          📅 {day.dt_txt} — 🌡 {day.main.temp}°C, {day.weather[0].description}
        </li>
      {/each}
    </ul>

    <h3>Forecast Chart</h3>
    <div style="width: 100%; max-width: 600px; height: 300px;">
      <canvas bind:this={chartCanvas}></canvas>
    </div>
  {/if}
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 2rem;
    padding: 1rem;
  }

  form {
    margin: 1rem 0;
  }

  input {
    padding: 0.5rem;
    font-size: 1rem;
    margin-right: 0.5rem;
  }

  button {
    padding: 0.5rem 1rem;
    font-size: 1rem;
  }

  ul {
    list-style-type: none;
    padding: 0;
    margin-top: 1rem;
  }

  li {
    margin-bottom: 0.5rem;
  }

  div {
    text-align: center;
    margin-top: 1rem;
  }
</style>