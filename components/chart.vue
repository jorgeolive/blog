  <template>
    <client-only>
        <line-chart :chartData="chartData" :chart-options="options"></line-chart>
    </client-only>
  </template>

<script>
  export default {
  name: "chart",
    props: {
      title: String,
      series: Array,
      labels: Array,
      xLabel: String,
      yLabel: String
    },
    data() {
      let defaultColors = ['rgba(16, 65, 129, 0.77)','rgba(56, 45, 129, 0.77)', 'rgba(426, 25, 129, 0.77)','rgba(124,252,0)'];
      return {
        options: {
          plugins: {
            title: {
                display: true,
                text: this.title
            }
        },
          scales: {
            yAxis: {
              title: {
                display: true,
                axis: 'y',
                text: this.yLabel
              }
            },
            xAxis: {
                title: {
                  display: true,
                  axis: 'x',
                  text: this.xLabel
                }
              }
            }
        },
        chartData: {
          datasets:
          this.series.map((x, index) =>{return  {
            label: x.label,
            data: x.data,
            backgroundColor: defaultColors[index],
            borderColor: defaultColors[index]
          }}), 
          labels: this.labels
        }
      };
    },
  }
  </script>
  