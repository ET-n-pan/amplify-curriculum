# day 14: 売上ページ作成

## ゴール
- 売上ページを作成する
- Chart.jsを使ってグラフを表示する
- APIから売上データを取得して表示する

## Chart.jsのインストール
Chart.jsを使ってチャートを表示します。以下のコマンドでインストールしてください。
```
npm install chart.js@latest
```

## 売上ページの作成
売上ページを作成します。
実際の動作確認は以下のURLから行ってください。カリキュラムを通して作成したアプリケーションがデプロイされています。

```
https://master.d3266wmyg8cwr5.amplifyapp.com
```
![売上ページ](../images/screenshots/day14-sales-page.png)

図表はChart.jsを使って表示します。図表コンポーネントのコードは提供しますが、適度の場所に配置してください。
```
<!-- components/SalesChart.vue -->
<template>
  <div>
    <!-- Chart Configuration -->
    <ui5-form>
      <ui5-form-item>
        <ui5-label slot="labelContent">チャートタイプ:</ui5-label>
        <ui5-select v-model="localConfig.type" @change="updateChart">
          <ui5-option value="bar">棒グラフ</ui5-option>
          <ui5-option value="line">折れ線グラフ</ui5-option>
          <ui5-option value="pie">円グラフ</ui5-option>
          <ui5-option value="doughnut">ドーナツグラフ</ui5-option>
          <ui5-option value="area">エリアグラフ</ui5-option>
          <ui5-option value="radar">レーダーチャート</ui5-option>
        </ui5-select>
      </ui5-form-item>

      <ui5-form-item>
        <ui5-label slot="labelContent">データ軸:</ui5-label>
        <ui5-select v-model="localConfig.dataAxis" @change="updateChart">
          <ui5-option value="monthly">月別売上</ui5-option>
          <ui5-option value="product">商品別売上</ui5-option>
          <ui5-option value="customer">顧客別売上</ui5-option>
          <ui5-option value="status">ステータス別</ui5-option>
          <ui5-option value="weekly">週別売上</ui5-option>
          <ui5-option value="payment">支払状況別</ui5-option>
        </ui5-select>
      </ui5-form-item>

      <ui5-form-item>
        <ui5-label slot="labelContent">集計方法:</ui5-label>
        <ui5-select v-model="localConfig.aggregation" @change="updateChart">
          <ui5-option value="sum">合計金額</ui5-option>
          <ui5-option value="count">件数</ui5-option>
          <ui5-option value="avg">平均金額</ui5-option>
          <ui5-option value="max">最大値</ui5-option>
          <ui5-option value="min">最小値</ui5-option>
        </ui5-select>
      </ui5-form-item>
    </ui5-form>

    <!-- Chart Display Area -->
    <div :style="{ position: 'relative', height: height + 'px', marginTop: '20px' }">
      <canvas :id="chartId"></canvas>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, reactive, nextTick } from 'vue';
import Chart from 'chart.js/auto';

// Propsを使って親コンポーネントからデータと設定を受け取る
const props = defineProps<{
  data: any[];
  height?: number;
  chartId?: string;
  config?: {
    type?: string;
    dataAxis?: string;
    aggregation?: string;
  };
}>();

// デフォルトのタイプと軸の設定
const localConfig = reactive({
  type: props.config?.type || 'bar',
  dataAxis: props.config?.dataAxis || 'monthly',
  aggregation: props.config?.aggregation || 'sum'
});

// チャートの初期化
let chartInstance: Chart | null = null;

// カラーパレット
const colorPalette = [
  { bg: 'rgba(255, 99, 132, 0.5)', border: 'rgba(255, 99, 132, 1)' },    // Red
  { bg: 'rgba(54, 162, 235, 0.5)', border: 'rgba(54, 162, 235, 1)' },   // Blue
  { bg: 'rgba(255, 206, 86, 0.5)', border: 'rgba(255, 206, 86, 1)' },   // Yellow
  { bg: 'rgba(75, 192, 192, 0.5)', border: 'rgba(75, 192, 192, 1)' },   // Teal
  { bg: 'rgba(153, 102, 255, 0.5)', border: 'rgba(153, 102, 255, 1)' }, // Purple
  { bg: 'rgba(255, 159, 64, 0.5)', border: 'rgba(255, 159, 64, 1)' },   // Orange
  { bg: 'rgba(199, 199, 199, 0.5)', border: 'rgba(199, 199, 199, 1)' }, // Grey
  { bg: 'rgba(83, 102, 255, 0.5)', border: 'rgba(83, 102, 255, 1)' },   // Indigo
  { bg: 'rgba(255, 99, 255, 0.5)', border: 'rgba(255, 99, 255, 1)' },   // Pink
  { bg: 'rgba(99, 255, 132, 0.5)', border: 'rgba(99, 255, 132, 1)' },   // Green
  { bg: 'rgba(255, 193, 7, 0.5)', border: 'rgba(255, 193, 7, 1)' },     // Amber
  { bg: 'rgba(0, 188, 212, 0.5)', border: 'rgba(0, 188, 212, 1)' },     // Cyan
  { bg: 'rgba(96, 125, 139, 0.5)', border: 'rgba(96, 125, 139, 1)' },   // Blue Grey
  { bg: 'rgba(156, 39, 176, 0.5)', border: 'rgba(156, 39, 176, 1)' },   // Deep Purple
  { bg: 'rgba(121, 85, 72, 0.5)', border: 'rgba(121, 85, 72, 1)' },     // Brown
];

// データの色を生成
const getColors = (count: number) => {
  const colors = { backgrounds: [] as string[], borders: [] as string[] };
  for (let i = 0; i < count; i++) {
    const color = colorPalette[i % colorPalette.length];
    colors.backgrounds.push(color.bg);
    colors.borders.push(color.border);
  }
  return colors;
};

// ラベル変換関数
const getStatusLabel = (status: string) => {
  const labels: Record<string, string> = {
    pending: '保留中',
    confirmed: '確定',
    shipped: '出荷済',
    delivered: '配達済'
  };
  return labels[status] || status;
};

// 支払状況ラベル変換関数
const getPaymentLabel = (status: string) => {
  const labels: Record<string, string> = {
    unpaid: '未払い',
    paid: '支払済',
    refunded: '返金済'
  };
  return labels[status] || status;
};

// 最初にチャートを初期化し、データをセット
onMounted(() => {
  initChart();
  updateChart();
});

// データの変更を監視し、チャートを更新
watch(() => props.data, () => {
  updateChart();
}, { deep: true });

// チャートの初期化
const initChart = () => {
  const ctx = document.getElementById(props.chartId || 'salesChart') as HTMLCanvasElement;
  if (ctx) {
    chartInstance = new Chart(ctx, {
      type: 'bar',
      data: { labels: [], datasets: [] },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { 
            position: 'top',
            labels: {
              padding: 20,
              font: { size: 12 }
            }
          },
          title: { 
            display: true, 
            text: '売上分析',
            font: { size: 16 }
          },
          tooltip: {
            callbacks: {
              label: (context) => {
                const label = context.dataset.label || '';
                const value = context.parsed.y || context.parsed;
                if (localConfig.aggregation === 'sum' || localConfig.aggregation === 'avg') {
                  return `${label}: ¥${value.toLocaleString()}`;
                }
                return `${label}: ${value}`;
              }
            }
          }
        }
      }
    });
  }
};

// チャートの更新
const updateChart = async () => {
  if (!chartInstance) return;
  await nextTick();
  const chartData = prepareChartData();
  chartInstance.data = chartData;
  
  // Handle chart type change
  if (localConfig.type === 'area') {
    chartInstance.config.type = 'line';
    chartInstance.data.datasets.forEach(dataset => {
      dataset.fill = true;
    });
  } else {
    chartInstance.config.type = localConfig.type as any;
  }

  // 特殊な処理が必要なチャートタイプ
  if (['pie', 'doughnut', 'radar'].includes(localConfig.type)) {
    chartInstance.options!.scales = {};
  } else {
    chartInstance.options!.scales = {
      y: { 
        beginAtZero: true,
        ticks: {
          callback: function(value) {
            if (localConfig.aggregation === 'sum' || localConfig.aggregation === 'avg') {
              return '¥' + value.toLocaleString();
            }
            return value;
          }
        }
      }
    };
  }
  
  chartInstance.update();
};

// チャートごとにデータを準備
const prepareChartData = () => {
  switch (localConfig.dataAxis) {
    case 'monthly':
      return prepareMonthlyData();
    case 'product':
      return prepareProductData();
    case 'customer':
      return prepareCustomerData();
    case 'status':
      return prepareStatusData();
    case 'weekly':
      return prepareWeeklyData();
    case 'payment':
      return preparePaymentData();
    default:
      return { labels: [], datasets: [] };
  }
};

// Data preparation functions
const prepareMonthlyData = () => {
  const monthlyData: Record<string, number[]> = {};
  
  props.data.forEach(sale => {
    const key = `${sale.sales_year}/${sale.sales_month}`;
    if (!monthlyData[key]) monthlyData[key] = [];
    monthlyData[key].push(sale.total_amount);
  });

  const labels = Object.keys(monthlyData).sort();
  const data = labels.map(key => aggregateValues(monthlyData[key]));
  const colors = getColors(labels.length);

  return {
    labels,
    datasets: [{
      label: getAggregationLabel(),
      data,
      backgroundColor: localConfig.type === 'line' ? colors.borders[0] : colors.backgrounds,
      borderColor: colors.borders,
      borderWidth: localConfig.type === 'line' ? 2 : 1,
      tension: 0.3
    }]
  };
};

// 商品別データ準備
const prepareProductData = () => {
  const productData: Record<string, number[]> = {};
  
  props.data.forEach(sale => {
    const key = sale.product_name || sale.product_code;
    if (!productData[key]) productData[key] = [];
    productData[key].push(sale.total_amount);
  });

  const labels = Object.keys(productData);
  const data = labels.map(key => aggregateValues(productData[key]));
  const colors = getColors(labels.length);

  return {
    labels,
    datasets: [{
      label: getAggregationLabel(),
      data,
      backgroundColor: colors.backgrounds,
      borderColor: colors.borders,
      borderWidth: 1
    }]
  };
};

// 顧客別データ準備
const prepareCustomerData = () => {
  const customerData: Record<string, number[]> = {};
  
  props.data.forEach(sale => {
    if (!customerData[sale.customer_code]) customerData[sale.customer_code] = [];
    customerData[sale.customer_code].push(sale.total_amount);
  });

  const labels = Object.keys(customerData);
  const data = labels.map(key => aggregateValues(customerData[key]));
  const colors = getColors(labels.length);

  return {
    labels,
    datasets: [{
      label: getAggregationLabel(),
      data,
      backgroundColor: colors.backgrounds,
      borderColor: colors.borders,
      borderWidth: 1
    }]
  };
};

// ステータス別データ準備
const prepareStatusData = () => {
  const statusData: Record<string, number[]> = {};
  
  props.data.forEach(sale => {
    const key = getStatusLabel(sale.order_status);
    if (!statusData[key]) statusData[key] = [];
    statusData[key].push(sale.total_amount);
  });

  const labels = Object.keys(statusData);
  const data = labels.map(key => aggregateValues(statusData[key]));
  const colors = getColors(labels.length);

  return {
    labels,
    datasets: [{
      label: getAggregationLabel(),
      data,
      backgroundColor: colors.backgrounds,
      borderColor: colors.borders,
      borderWidth: 1
    }]
  };
};

// 週別データ準備
const prepareWeeklyData = () => {
  const weeklyData: Record<string, number[]> = {};
  
  props.data.forEach(sale => {
    if (sale.sales_week) {
      const key = `${sale.sales_year} W${sale.sales_week}`;
      if (!weeklyData[key]) weeklyData[key] = [];
      weeklyData[key].push(sale.total_amount);
    }
  });

  const labels = Object.keys(weeklyData).sort();
  const data = labels.map(key => aggregateValues(weeklyData[key]));
  const colors = getColors(labels.length);

  return {
    labels,
    datasets: [{
      label: getAggregationLabel(),
      data,
      backgroundColor: colors.backgrounds,
      borderColor: colors.borders,
      borderWidth: 1
    }]
  };
};

// 支払状況別データ準備
const preparePaymentData = () => {
  const paymentData: Record<string, number[]> = {};
  
  props.data.forEach(sale => {
    const key = getPaymentLabel(sale.payment_status);
    if (!paymentData[key]) paymentData[key] = [];
    paymentData[key].push(sale.total_amount);
  });

  const labels = Object.keys(paymentData);
  const data = labels.map(key => aggregateValues(paymentData[key]));
  const colors = getColors(labels.length);

  return {
    labels,
    datasets: [{
      label: getAggregationLabel(),
      data,
      backgroundColor: colors.backgrounds,
      borderColor: colors.borders,
      borderWidth: 1
    }]
  };
};

// 値の集約
const aggregateValues = (values: number[]): number => {
  if (values.length === 0) return 0;
  
  switch (localConfig.aggregation) {
    case 'sum':
      return values.reduce((a, b) => a + b, 0);
    case 'count':
      return values.length;
    case 'avg':
      return values.reduce((a, b) => a + b, 0) / values.length;
    case 'max':
      return Math.max(...values);
    case 'min':
      return Math.min(...values);
    default:
      return 0;
  }
};

// 集約ラベルの取得
const getAggregationLabel = (): string => {
  const labels: Record<string, string> = {
    sum: '売上金額',
    count: '件数',
    avg: '平均金額',
    max: '最大金額',
    min: '最小金額'
  };
  return labels[localConfig.aggregation] || '値';
};
</script>
```

使い方の例:
```
<!-- src/pages/SalesPage.vue -->
<SalesChart 
:data="salesStore.filteredSales"
:height="400"
chartId="mainSalesChart"
/>
```
dataは売上データの配列を渡します。
