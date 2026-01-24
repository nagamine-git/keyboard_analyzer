<script>
  import { onMount, afterUpdate } from 'svelte';
  import {
    Chart,
    RadarController,
    RadialLinearScale,
    PointElement,
    LineElement,
    Filler,
    Tooltip,
    Legend
  } from 'chart.js';

  // Chart.jsに必要なコンポーネントを登録
  Chart.register(
    RadarController,
    RadialLinearScale,
    PointElement,
    LineElement,
    Filler,
    Tooltip,
    Legend
  );

  export let data;

  let canvasContainer;
  let canvas;
  let chart;

  const colors = [
    { bg: 'rgba(255, 99, 132, 0.2)', border: 'rgba(255, 99, 132, 1)', point: 'rgba(255, 99, 132, 1)' },
    { bg: 'rgba(54, 162, 235, 0.2)', border: 'rgba(54, 162, 235, 1)', point: 'rgba(54, 162, 235, 1)' },
    { bg: 'rgba(75, 192, 192, 0.2)', border: 'rgba(75, 192, 192, 1)', point: 'rgba(75, 192, 192, 1)' },
    { bg: 'rgba(255, 206, 86, 0.2)', border: 'rgba(255, 206, 86, 1)', point: 'rgba(255, 206, 86, 1)' },
    { bg: 'rgba(153, 102, 255, 0.2)', border: 'rgba(153, 102, 255, 1)', point: 'rgba(153, 102, 255, 1)' },
  ];

  function createChart() {
    if (chart) {
      chart.destroy();
      chart = null;
    }

    // キャンバスを再作成してサイズの問題を解決
    if (canvas) {
      const newCanvas = document.createElement('canvas');
      canvasContainer.innerHTML = '';
      canvasContainer.appendChild(newCanvas);
      canvas = newCanvas;
    }

    const ctx = canvas.getContext('2d');

    // データセットの配列を作成
    let datasets;
    if (Array.isArray(data.datasets)) {
      // 複数のデータセットが渡された場合
      datasets = data.datasets.map((ds, i) => ({
        label: ds.label,
        data: ds.values,
        backgroundColor: colors[i % colors.length].bg,
        borderColor: colors[i % colors.length].border,
        borderWidth: 2,
        pointBackgroundColor: colors[i % colors.length].point,
        pointBorderColor: '#fff',
        pointHoverBackgroundColor: '#fff',
        pointHoverBorderColor: colors[i % colors.length].border
      }));
    } else {
      // 単一データセットの場合（後方互換性）
      datasets = [{
        label: data.label || 'スコア',
        data: data.values,
        backgroundColor: colors[0].bg,
        borderColor: colors[0].border,
        borderWidth: 2,
        pointBackgroundColor: colors[0].point,
        pointBorderColor: '#fff',
        pointHoverBackgroundColor: '#fff',
        pointHoverBorderColor: colors[0].border
      }];
    }

    chart = new Chart(ctx, {
      type: 'radar',
      data: {
        labels: data.labels,
        datasets: datasets
      },
      options: {
        responsive: true,
        maintainAspectRatio: true,
        aspectRatio: 1,
        scales: {
          r: {
            min: 0,
            max: 100,
            ticks: {
              stepSize: 20
            }
          }
        },
        plugins: {
          legend: {
            display: datasets.length > 1,
            position: 'bottom'
          }
        }
      }
    });
  }

  onMount(() => {
    if (data && canvasContainer) {
      canvas = canvasContainer.querySelector('canvas');
      createChart();
    }
  });

  afterUpdate(() => {
    if (data && canvasContainer) {
      createChart();
    }
  });
</script>

<div bind:this={canvasContainer} style="position: relative; width: 100%; height: 400px;">
  <canvas></canvas>
</div>
