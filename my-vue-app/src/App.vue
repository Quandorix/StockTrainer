
<template>
  <div class="container">
    <header class="header">
      <h1>Aktien Trading Trainer</h1>
      <p>Üben Sie den Aktienhandel mit simulierten Kursdaten und virtuellem Geld</p>
    </header>

    <section class="dashboard">
      <div class="card">
        <h3>💰 Virtuelles Kapital</h3>
        <div class="value">€{{ balance.toFixed(2) }}</div>
      </div>
      <div class="card">
        <h3>📊 Portfolio Wert</h3>
        <div class="value">€{{ portfolioValue.toFixed(2) }}</div>
      </div>
      <div class="card">
        <h3>📈 Gesamt Performance</h3>
        <div class="value" :class="{ profit: performance >= 0, loss: performance < 0 }">
          {{ performance >= 0 ? '+' : '' }}{{ performance.toFixed(2) }}%
        </div>
      </div>
    </section>

    <section class="trading-section">
      <div class="trading-panel">
        <h3>🛒 Aktien Handel</h3>
        <div class="form-group">
          <label>Startkapital festlegen:</label>
          <input type="number" v-model.number="startBalance" min="1000">
          <button class="btn" @click="setStartBalance">Kapital festlegen</button>
        </div>
        <div class="form-group">
          <label>Aktien-Symbol:</label>
          <select v-model="currentStock">
            <option value="" disabled>Aktie wählen...</option>
            <option v-for="(stock, symbol) in stockOptions" :key="symbol" :value="symbol">
              {{ stock.label }}
            </option>
          </select>
        </div>
        <div class="stock-price" v-if="currentStock && stockPrices[currentStock]?.price">
          Aktueller Preis: ${{ stockPrices[currentStock].price.toFixed(2) }}
        </div>
        <div class="form-group">
          <label>Anzahl Aktien:</label>
          <input type="number" v-model.number="quantity" min="1">
        </div>
        <button class="btn" @click="buyStock">📈 Kaufen</button>
        <button class="btn btn-sell" @click="sellStock">📉 Verkaufen</button>
        <div v-if="message.text" :class="message.type">{{ message.text }}</div>
      </div>
      <Chart :currentStock="currentStock" :stockPrices="stockPrices" />
    </section>

    <section class="portfolio">
      <h3>💼 Mein Portfolio</h3>
      <div v-if="Object.keys(portfolio).length === 0" class="loading">
        Noch keine Aktien im Portfolio
      </div>
      <div v-else v-for="(holding, symbol) in portfolio" :key="symbol" class="stock-item">
        <div class="stock-info">
          <div class="stock-symbol">{{ symbol }}</div>
          <div class="stock-details">
            {{ holding.shares }} Aktien @ Ø ${{ holding.avgPrice.toFixed(2) }}
          </div>
        </div>
        <div class="stock-performance">
          <div>${{ (holding.shares * stockPrices[symbol].price).toFixed(2) }}</div>
          <div :class="{ profit: getProfit(symbol) >= 0, loss: getProfit(symbol) < 0 }">
            {{ getProfit(symbol) >= 0 ? '+' : '' }}${{ getProfit(symbol).toFixed(2) }}
            ({{ getProfitPercent(symbol) >= 0 ? '+' : '' }}{{ getProfitPercent(symbol).toFixed(2) }}%)
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import Chart from './Chart.vue'; 

export default {
  name: 'App',
  components: { Chart },
  setup() {
    // State
    const balance = ref(10000);
    const startBalance = ref(10000);
    const portfolio = ref({});
    const currentStock = ref('');
    const quantity = ref(1);
    const message = ref({ text: '', type: '' });

    // Stock options
    const stockOptions = {
      'AAPL': { label: 'Apple (AAPL)' },
      'GOOGL': { label: 'Alphabet (GOOGL)' },
      'MSFT': { label: 'Microsoft (MSFT)' },
      'AMZN': { label: 'Amazon (AMZN)' },
      'TSLA': { label: 'Tesla (TSLA)' },
      'META': { label: 'Meta (META)' },
      'NVDA': { label: 'NVIDIA (NVDA)' },
      'JPM': { label: 'JPMorgan Chase (JPM)' },
      'JNJ': { label: 'Johnson & Johnson (JNJ)' },
      'V': { label: 'Visa (V)' },
      'SAP': { label: 'SAP (SAP)' },
      'ASML': { label: 'ASML (ASML)' }
    };

    // Mock stock prices and historical data
    const stockPrices = ref({
      'AAPL': { price: 192.25, history: generateMockHistory(192.25) },
      'GOOGL': { price: 187.56, history: generateMockHistory(187.56) },
      'MSFT': { price: 429.04, history: generateMockHistory(429.04) },
      'AMZN': { price: 194.35, history: generateMockHistory(194.35) },
      'TSLA': { price: 468.83, history: generateMockHistory(468.83) },
      'META': { price: 669.74, history: generateMockHistory(669.74) },
      'NVDA': { price: 135.58, history: generateMockHistory(135.58) },
      'JPM': { price: 237.55, history: generateMockHistory(237.55) },
      'JNJ': { price: 148.75, history: generateMockHistory(148.75) },
      'V': { price: 305.77, history: generateMockHistory(305.77) },
      'SAP': { price: 222.53, history: generateMockHistory(222.53) },
      'ASML': { price: 824.92, history: generateMockHistory(824.92) }
    });

    // Generate mock historical data (30 days)
    function generateMockHistory(basePrice) {
      const history = [];
      const today = new Date('2025-06-04');
      for (let i = 0; i < 30; i++) {
        const date = new Date(today);
        date.setDate(today.getDate() - i);
        // Simulate price fluctuations (±5% of base price)
        const fluctuation = basePrice * (Math.random() * 0.1 - 0.05);
        history.push({
          date,
          price: parseFloat((basePrice + fluctuation).toFixed(2))
        });
      }
      return history.reverse();
    }

    // Computed values
    const portfolioValue = computed(() => {
      return Object.keys(portfolio.value).reduce((total, symbol) => {
        if (stockPrices.value[symbol]) {
          return total + portfolio.value[symbol].shares * stockPrices.value[symbol].price;
        }
        return total;
      }, 0);
    });

    const performance = computed(() => {
      const totalValue = balance.value + portfolioValue.value;
      return ((totalValue - startBalance.value) / startBalance.value * 100);
    });

    // Simulate price updates for all stocks
    let priceUpdateInterval = null;
    function startPriceUpdates() {
      priceUpdateInterval = setInterval(() => {
        Object.keys(stockPrices.value).forEach(symbol => {
          const currentPrice = stockPrices.value[symbol].price;
          const fluctuation = currentPrice * (Math.random() * 0.04 - 0.02); // ±2% fluctuation
          stockPrices.value[symbol].price = parseFloat((currentPrice + fluctuation).toFixed(2));
        });
        // Trigger reactivity
        stockPrices.value = { ...stockPrices.value };
      }, 60000); // Update every 60 seconds
    }

    // Methoden
    function setStartBalance() {
      if (startBalance.value >= 1000) {
        balance.value = startBalance.value;
        portfolio.value = {};
        showMessage('Startkapital erfolgreich festgelegt!', 'success');
      } else {
        showMessage('Mindestkapital: €1,000', 'error');
      }
    }

    function buyStock() {
      const symbol = currentStock.value;
      const qty = quantity.value;

      if (!symbol || !qty || qty <= 0) {
        showMessage('Bitte wählen Sie eine Aktie und geben Sie eine gültige Anzahl ein.', 'error');
        return;
      }

      const stock = stockPrices.value[symbol];
      if (!stock || !stock.price) {
        showMessage('Keine Preisdaten verfügbar!', 'error');
        return;
      }

      const totalCost = stock.price * qty;

      if (totalCost > balance.value) {
        showMessage('Nicht genügend Kapital verfügbar!', 'error');
        return;
      }

      balance.value -= totalCost;

      if (!portfolio.value[symbol]) {
        portfolio.value[symbol] = { shares: 0, avgPrice: 0, totalInvested: 0 };
      }

      portfolio.value[symbol].totalInvested += totalCost;
      portfolio.value[symbol].shares += qty;
      portfolio.value[symbol].avgPrice = portfolio.value[symbol].totalInvested / portfolio.value[symbol].shares;

      showMessage(`${qty} ${symbol} Aktien für $${totalCost.toFixed(2)} gekauft!`, 'success');
    }

    function sellStock() {
      const symbol = currentStock.value;
      const qty = quantity.value;

      if (!symbol || !qty || qty <= 0) {
        showMessage('Bitte wählen Sie eine Aktie und geben Sie eine gültige Anzahl ein.', 'error');
        return;
      }

      if (!portfolio.value[symbol] || portfolio.value[symbol].shares < qty) {
        showMessage('Nicht genügend Aktien im Portfolio!', 'error');
        return;
      }

      const stock = stockPrices.value[symbol];
      if (!stock || !stock.price) {
        showMessage('Keine Preisdaten verfügbar!', 'error');
        return;
      }

      const totalRevenue = stock.price * qty;

      balance.value += totalRevenue;

      portfolio.value[symbol].shares -= qty;
      portfolio.value[symbol].totalInvested -= portfolio.value[symbol].avgPrice * qty;

      if (portfolio.value[symbol].shares === 0) {
        delete portfolio.value[symbol];
      }

      showMessage(`${qty} ${symbol} Aktien für $${totalRevenue.toFixed(2)} verkauft!`, 'success');
    }

    function showMessage(text, type) {
      message.value = { text, type };
      setTimeout(() => {
        message.value = { text: '', type: '' };
      }, 3000);
    }

    function getProfit(symbol) {
      const holding = portfolio.value[symbol];
      const currentValue = holding.shares * stockPrices.value[symbol].price;
      return currentValue - holding.totalInvested;
    }

    function getProfitPercent(symbol) {
      const holding = portfolio.value[symbol];
      const profit = getProfit(symbol);
      return (profit / holding.totalInvested) * 100;
    }

    onMounted(() => {
      // Start price updates for all stocks
      startPriceUpdates();
    });

    onUnmounted(() => {
      // Clear interval when component is unmounted
      if (priceUpdateInterval) {
        clearInterval(priceUpdateInterval);
      }
    });

    return {
      balance,
      startBalance,
      portfolio,
      currentStock,
      quantity,
      message,
      stockOptions,
      stockPrices,
      portfolioValue,
      performance,
      setStartBalance,
      buyStock,
      sellStock,
      getProfit,
      getProfitPercent
    };
  }
}
</script>

<style scoped>
/* Scoped styles can be added here if needed, but we'll rely on style.css */
</style>
