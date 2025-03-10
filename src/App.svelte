<!-- src/App.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';

  // Define types
  type Currency ={
    code: string;
    value: number;
    name: string;
  }

  type CurrencyData = {
    [key: string]: number;
  }

  type CurrencyNames = {
    [key: string]: string;
  }

  // Import currencyNames at module level with proper typing
  import currencyNamesImport from './assets/currencies.json';
  // Explicitly type the imported JSON
  const currencyNames: CurrencyNames = currencyNamesImport as CurrencyNames;

  // Data untuk menyimpan kurs mata uang
  let currencies: CurrencyData = {};
  let currencyList: Currency[] = [];
  let loading: boolean = true;
  let error: string | null = null;
  let darkMode: boolean = false;

  // Data untuk kalkulator
  let selectedCurrency: string = 'idr';
  let amount: number = 1;
  let conversionType: 'fromUSD' | 'toUSD' = 'fromUSD'; // 'fromUSD' atau 'toUSD'
  let result: number = 0;

  // Fungsi untuk mengambil data mata uang dari API
  const fetchCurrencyRates = async () => {
    try {
      const ratesResponse = await fetch('https://latest.currency-api.pages.dev/v1/currencies/usd.json');
      const ratesData = await ratesResponse.json();

      // Data dari API berisi nilai tukar dari USD ke mata uang lain
      currencies = ratesData.usd || {};

      // Buat array dari objek mata uang untuk dropdown
      currencyList = Object.keys(currencies)
      .filter(code => Object.prototype.hasOwnProperty.call(currencyNames, code)) // Safer check
      .map(code => ({
        code,
        value: currencies[code],
        name: currencyNames[code]
      }))
      .sort((a, b) => a.code.localeCompare(b.code));

      // Set default currency ke IDR jika tersedia
      const idrCurrency = currencyList.find(c => c.code === 'idr');
      if (idrCurrency) {
        selectedCurrency = 'idr';
      } else if (currencyList.length > 0) {
        selectedCurrency = currencyList[0].code;
      }

      calculateConversion();
      loading = false;
    } catch (err) {
      console.error('Error fetching currency rates:', err);
      error = 'Gagal mengambil data kurs. Silakan coba lagi nanti.';
      loading = false;
    }
  };

  // Fungsi untuk menghitung konversi mata uang
  const calculateConversion = (): void => {
  // Pastikan amount adalah angka valid dan lebih dari 0
  const validAmount = amount && !isNaN(amount) ? amount : 0;
  const rate = currencies[selectedCurrency] ?? 0;

  if (validAmount > 0 && rate > 0) {
    result = conversionType === 'fromUSD'
      ? validAmount * rate
      : validAmount / rate;
  } else {
    result = 0;
  }
};

  // Toggle dark mode
  const toggleDarkMode = () => {
    darkMode = !darkMode;
    localStorage.theme = darkMode ? 'dark' : 'light';
    document.documentElement.classList.toggle('dark', darkMode);
  };

  // Jalankan fungsi fetch saat komponen dimount
  onMount(() => {
    fetchCurrencyRates();

    // Muat preferensi tema dari localStorage atau deteksi preferensi sistem
    darkMode = localStorage.theme === 'dark' || (!('theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches);
    document.documentElement.classList.toggle('dark', darkMode);
  });
</script>

<div class="min-h-screen transition-colors duration-300 bg-gray-50 dark:bg-gray-900 text-gray-800 dark:text-gray-200">
  <div class="container mx-auto px-4 py-8">
    <div class="flex justify-between items-center mb-8">
      <h1 class="text-3xl font-bold text-center dark:text-white">Konverter Mata Uang</h1>

      <button
        on:click={toggleDarkMode}
        class="p-2 rounded-full dark:bg-gray-700 dark:text-yellow-300 bg-gray-200 text-gray-700 cursor-pointer"
      >
        {#if darkMode}
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" />
          </svg>
        {:else}
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" />
          </svg>
        {/if}
      </button>
    </div>

    {#if loading}
      <div class="flex justify-center items-center py-8">
        <div class="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 dark:border-primary border-secondary"></div>
      </div>
    {:else if error}
      <div class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4 mb-6 rounded dark:bg-red-900 dark:text-red-200">
        <p>{error}</p>
      </div>
    {:else}
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 items-start">

        <!-- Kurs Mata Uang -->
        <div class="bg-white dark:bg-gray-800 rounded-lg shadow-lg p-6 border-2 border-gray-100 dark:border-0">
          <h2 class="text-xl font-semibold mb-4 dark:text-white">Kurs Mata Uang terhadap USD</h2>
          <div class="overflow-x-auto">
            <table class="min-w-full divide-y divide-gray-200 dark:divide-gray-700">
              <thead>
                <tr>
                  <th class="px-4 py-3 text-left font-medium dark:text-gray-300">Kode</th>
                  <th class="px-4 py-3 text-left font-medium dark:text-gray-300">Mata Uang</th>
                  <th class="px-4 py-3 text-right font-medium dark:text-gray-300">Nilai</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-100 dark:divide-gray-700">
                {#each currencyList as currency}
                  {#if currency.code !== 'usd'}
                    <tr class="hover:bg-gray-50 dark:hover:bg-gray-700">
                      <td class="px-4 py-3 font-medium">{currency.code.toUpperCase()}</td>
                      <td class="px-4 py-3">{currency.name}</td>
                      <td class="px-4 py-3 text-right">{currency.value.toFixed(6)}</td>
                    </tr>
                  {/if}
                {/each}
              </tbody>
            </table>
          </div>
        </div>

        <!-- Kalkulator -->
        <div class="bg-white dark:bg-gray-800 rounded-lg shadow-lg p-6 border-2 border-gray-100 dark:border-0">
          <h2 class="text-xl font-semibold mb-4 dark:text-white">Kalkulator Nilai Tukar</h2>
          <div class="space-y-4">
            <div>
              <label for="amount" class="block mb-1 font-medium dark:text-gray-300">Jumlah:</label>
              <input
                type="number"
                id="amount"
                bind:value={amount}
                on:input={() => {
                  amount = Math.max(1, Number(amount) || 1); // Pastikan minimal 1
                  calculateConversion();
                }}
                min="1"
                class="w-full p-2 border border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600 dark:text-white"
              >
            </div>

            <div>
              <label for="conversionType" class="block mb-1 font-medium dark:text-gray-300">Jenis Konversi:</label>
              <select
                id="conversionType"
                bind:value={conversionType}
                on:change={calculateConversion}
                class="w-full p-2 border border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600 dark:text-white"
              >
                <option value="fromUSD">USD ke Mata Uang Lain</option>
                <option value="toUSD">Mata Uang Lain ke USD</option>
              </select>
            </div>

            <div>
              <label for="currency" class="block mb-1 font-medium dark:text-gray-300">Mata Uang:</label>
              <select
                id="currency"
                bind:value={selectedCurrency}
                on:change={calculateConversion}
                class="w-full p-2 border border-gray-300 rounded dark:bg-gray-700 dark:border-gray-600 dark:text-white"
              >
                {#each currencyList as currency}
                  {#if currency.code !== 'usd'}
                    <option value={currency.code}>{currency.code.toUpperCase()} - {currency.name}</option>
                  {/if}
                {/each}
              </select>
            </div>

            <div class="p-4 bg-gray-100 dark:bg-gray-700 rounded-lg mt-6">
              <h3 class="font-semibold mb-2 dark:text-white">Hasil Konversi:</h3>
              {#if conversionType === 'fromUSD'}
                <p class="text-lg">
                  {amount.toLocaleString('en-US')} USD =
                  <span class="font-bold">{result.toLocaleString('en-US')} {selectedCurrency.toUpperCase()}</span>
                </p>
              {:else}
                <p class="text-lg">
                  {amount.toLocaleString('en-US')} {selectedCurrency.toUpperCase()} =
                  <span class="font-bold">{result.toLocaleString('en-US')} USD</span>
                </p>
              {/if}
            </div>
          </div>
        </div>
      </div>

      <div class="mt-6 text-center text-sm text-gray-500 dark:text-gray-400">
        Data kurs diambil dari <a class="underline underline-offset-2" target="_blank" href="https://github.com/fawazahmed0/exchange-api?tab=readme-ov-file">exchange-api</a>. Terakhir diperbarui: {new Date().toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' })}
      </div>
    {/if}
  </div>
</div>
