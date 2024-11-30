
```dataviewjs
const options = {
  tickerPath: '"Finances/Repo/Orders"',
  table: {
    enabled: true,
    summary: {
      totalCost: true,
      portfolioValue: true,
      unrealizedGains: true,
      realizedGains: true,
      percentChange: true
    },
    columns: {
      ticker: true,
      shares: true,
      costBasis: true,
      mark: true,
      unrealizedGains: true,
      realizedGains: true,
      percentChange: true
    }
  },
  lineChart: {
    enabled: true,
    color: {
      r: 75,
      g: 192,
      b: 192,
    },
  },
  pieChart:{
    enabled: true,
  },
  barChart:{
    enabled: true,
    color: {
      r: 75,
      g: 192,
      b: 192,
    }
  }
}

// Variables
let tagPages = dv.pages(options.tickerPath);
const tableData = [];
let runningCost = 0;
let runningGains = 0;

let chartData = [];
let chartLabels = [];
let chartValues = {}

let pieChartData = {}

let barChart = {}
const barLabels = []
const barData = []

function getRandomColor() {
  const r = Math.floor(Math.random() * 256);
  const g = Math.floor(Math.random() * 256);
  const b = Math.floor(Math.random() * 256);
  return `rgba(${r}, ${g}, ${b}, 0.6)`;
}

function formatNumber(num) {
  num = Number(num);
  return Number(num.toFixed(2)).toLocaleString();
}

// Iterate all pages where ticker is file name
await Promise.all(tagPages.map(async note => {
  let shares = 0;
  let gains = 0;
  let profits = 0;
  let totalCost = 0;
  let percentIncrease = 0;
  const url = `https://query1.finance.yahoo.com/v8/finance/chart/${note.file.name}?range=7d&interval=1d`;
  const response = await requestUrl({
    url
  });

  // Build Table data
  if (options.table.enabled) {
    const marketPrice = response.json.chart.result[0].meta.regularMarketPrice;
    shares = note.Shares;
    profits = note.Profits;
    const costBasis = note["Cost Basis"];
    totalCost = shares * costBasis;
    gains = (shares * marketPrice) - totalCost;
    runningGains += gains + profits;
    runningCost += totalCost;
    percentIncrease = (gains / totalCost) * 100;

    let tableRow = [];
    if (options.table.columns.ticker) tableRow.push(note.file.name);
    if (options.table.columns.shares) tableRow.push(shares);
    if (options.table.columns.costBasis) tableRow.push(costBasis ? `$${formatNumber(costBasis)}` : '--');
    if (options.table.columns.mark) tableRow.push(`$${formatNumber(marketPrice)}`);
    if (options.table.columns.unrealizedGains) tableRow.push(gains ? `$${formatNumber(gains)}` : '--');
    if (options.table.columns.realizedGains) tableRow.push(profits ? `$${formatNumber(profits)}` : '--');
    if (options.table.columns.percentChange) tableRow.push(percentIncrease ? formatNumber(percentIncrease) + "%" : '--');

    tableData.push(tableRow);
  }

  // Build Pie Chart data
  if (options.pieChart.enabled) {
    pieChartData[note.file.name] = totalCost;
  }

  // Build Line Chart data
  if (options.lineChart.enabled) {
    const dates = response.json.chart.result[0].timestamp;
    const prices = response.json.chart.result[0].indicators.quote[0].close;
    dates.map((date, i) => {
      chartValues[date] ? chartValues[date] += (shares * prices[i]) + profits : chartValues[date] = (shares * prices[i]) + profits
    });
  }

  // Build Bar Chart data
  if (options.barChart.enabled) {
    barLabels.push(note.file.name)
    barData.push(percentIncrease)
  }
}));

// Display table results
if (options.table.enabled) {
  tableData.sort((a, b) => parseFloat(b[4].replace('$', '')) - parseFloat(a[4].replace('$', '')));

  const totalPortfolioValue = runningGains + runningCost;
  const percentIncrease = (runningGains / runningCost) * 100;
  dv.paragraph("---");
  dv.table([], [
    ["**Total Cost**", `$${formatNumber(runningCost)}`],
    ["**Portfolio Value**", `$${formatNumber(totalPortfolioValue)}`],
    ["**Unrealized Gains**", `$${formatNumber(runningGains)}`],
    ["**Percent Change**", `${formatNumber(percentIncrease)}%`]
  ]);
  dv.paragraph("---");
  let headers = []
  if (options.table.columns.ticker) headers.push("Ticker");
  if (options.table.columns.shares) headers.push("Shares");
  if (options.table.columns.costBasis) headers.push("Cost Basis");
  if (options.table.columns.mark) headers.push("Mark");
  if (options.table.columns.unrealizedGains) headers.push("Unrealized");
  if (options.table.columns.realizedGains) headers.push("Profits");
  if (options.table.columns.percentChange) headers.push("% Diff");

  dv.table(headers, tableData);
}

// Display chart data
if (options.lineChart.enabled) {
  chartLabels = Object.keys(chartValues).map(c => {
    return (new Date(c * 1000).getMonth() + 1) + "/" + (new Date(c * 1000).getDate());
  });
  chartData = Object.keys(chartValues).map(c => chartValues[c]);

  dv.paragraph("---");
  const lineChart = {
    type: 'line',
    data: {
      labels: chartLabels,
      datasets: [{
        label: '7d Portfolio Value',
        data: chartData,
        backgroundColor: `rgb(${options.lineChart.color.r}, ${options.lineChart.color.g}, ${options.lineChart.color.b}, 0.2)`,
        borderColor: `rgb(${options.lineChart.color.r}, ${options.lineChart.color.g}, ${options.lineChart.color.b}, 1)`,
        borderWidth: 2
      }]
    },
  };
  window.renderChart(lineChart, this.container);
}

// Display pie chart and bar chart side by side
if (options.pieChart.enabled || options.barChart.enabled) {
  dv.paragraph("---");

  // Create a container for the charts
  const chartContainer = dv.container.createEl('div', {
    cls: 'chart-container'
  });

  // Pie Chart
  if (options.pieChart.enabled) {
    const backgroundColors = Object.keys(pieChartData).map(() => getRandomColor());
    const pieChart = {
      type: 'pie',
      data: {
        labels: Object.keys(pieChartData),
        datasets: [{
          label: '7d Portfolio.',
          data: Object.keys(pieChartData).map(p => pieChartData[p]),
          backgroundColor: backgroundColors,
          borderColor: 'rgba(0, 0, 0, 1)',
          borderWidth: 2
        }]
      },
    };
    // Render the pie chart in the left container
    const pieChartContainer = chartContainer.createEl('div', {
      cls: 'chart pie-chart'
    });
    pieChartContainer.style.height = '400px'; // Set the height of the pie chart container
    window.renderChart(pieChart, pieChartContainer);
  }

  // Bar Chart
  if (options.barChart.enabled) {
    // Sort bar data by percent increase
    const sortedBarData = barLabels.map((label, index) => ({
        label,
        value: barData[index]
      }))
      .sort((a, b) => b.value - a.value);

    const sortedLabels = sortedBarData.map(item => item.label);
    const sortedValues = sortedBarData.map(item => item.value);

    const barChart = {
      type: 'bar',
      data: {
        labels: sortedLabels,
        datasets: [{
          label: 'Percent Gains',
          data: sortedValues,
          backgroundColor: `rgba(${options.barChart.color.r}, ${options.barChart.color.g}, ${options.barChart.color.b}, 0.6)`,
          borderColor: `rgba(${options.barChart.color.r}, ${options.barChart.color.g}, ${options.barChart.color.b}, 1)`,
          borderWidth: 1
        }]
      },
      options: {
        indexAxis: 'y', // Horizontal bar chart
        scales: {
          x: {
            beginAtZero: true,
            title: {
              display: true,
              text: 'Percent Gains (%)'
            }
          },
          y: {
            title: {
              display: true,
              text: 'Stocks'
            },
            ticks: {
              autoSkip: false, // Prevent skipping of labels
              maxRotation: 0,
              minRotation: 0,
            }
          }
        },
        maintainAspectRatio: false, // Allow custom height
      }
    };
    // Render the bar chart in the right container
    const barChartContainer = chartContainer.createEl('div', {
      cls: 'chart bar-chart'
    });
    barChartContainer.style.height = '400px'; // Set the height of the bar chart container
    window.renderChart(barChart, barChartContainer);
  }
}

// Add CSS for the chart layout
const style = document.createElement('style');
style.textContent = `
	.chart-container {
		display: flex;
		justify-content: space-between;
	}
	.chart {
		width: 48%; /* Adjust width as needed */
	}
	.pie-chart {
		margin-right: 10px; /* Optional: Add space between charts */
	}
`;
document.head.appendChild(style);
```
