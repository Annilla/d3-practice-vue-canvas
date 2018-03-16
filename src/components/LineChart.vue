<template>
  <div class="lineChart">
    <h1>折線圖</h1>
    <div class="detail">高雄市不動產買賣統計(104年6-10月)</div>
    <!-- 圖表 -->
    <div class="chartContain">
      <canvas class="chart"></canvas>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  data() {
    return {
      data: [], // From randomData()
      chart: {
        width: 500,
        height: 500,
        paddingTop: 30,
        paddingRight: 30,
        paddingBottom: 100,
        paddingLeft: 60
      },
      conWidth: undefined, // Get Container Width
      c: undefined, // Get Canvas
      ctx: undefined // Set Canvas 2D
    };
  },
  mounted() {
    // Init Canvas
    this.initCanvas();

    // Window Resize
    window.addEventListener("resize", this.initCanvas);
  },
  methods: {
    initCanvas() {
      let WHrate =
        (this.chart.width + this.chart.paddingLeft + this.chart.paddingRight) /
        (this.chart.height + this.chart.paddingTop + this.chart.paddingBottom);
      
      // Get Container Width
      this.conWidth = document.querySelector(".chartContain").offsetWidth;
      this.c = document.querySelector(".chart");
      this.ctx = this.c.getContext("2d");

      // Set Canvas Width
      this.c.width = this.conWidth;
      this.c.height = this.conWidth / WHrate;

      // Clear Canvas
      this.ctx.clearRect(0, 0, this.c.width, this.c.height);
      this.drawCanvas();
    },
    drawCanvas() {
      // Draw Canvas
      this.ctx.moveTo(0, 0);
      this.ctx.lineTo(this.c.width, this.c.height);
      this.ctx.stroke();
    }
  }
};
</script>

<style lang="postcss">
.lineChart {
  /* 說明 */
  .detail {
    color: gray;
  }
  .chartContain {
    max-width: 600px;
    margin: 0 auto;
    .chart {
      background-color: yellow;
    }
  }
}
</style>
