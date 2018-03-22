<template>
  <div class="barpChart">
    <h1>橫條圖</h1>
    <div class="detail">高雄市鼓山區不動產買賣統計(104年)</div>
    <!-- 圖表 -->
    <div class="chartContain">
      <div :class="{ tooltip: true, hidden: hideTooltip}">
        <div class="name">左營區 / 10月</div>
        <div class="value">214 件</div>
      </div>
    </div>
    <button @click="randomData">隨機資料</button>
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
      hideTooltip: true,
      conWidth: 0, // Get Container Width
      canvas: undefined, // For D3 Draw Canvas
      ctx: undefined, // Set Canvas 2D
      canvasA: undefined, // For D3 Draw Canvas Animate
      ctxA: undefined, // Set Canvas 2D Animate
      customBase: undefined, // This is the parent of all other elements
      custom: undefined,
      xTick: undefined, // X軸 tick
    };
  },
  computed: {
    // Caculate Container Height/Width Ratio
    conRate() {
      return (
        (this.chart.height + this.chart.paddingTop + this.chart.paddingBottom) /
        (this.chart.width + this.chart.paddingLeft + this.chart.paddingRight)
      );
    },
    // Caculate Container Height
    conHeight() {
      return this.conWidth * this.conRate;
    },
    // Caculate Chart Width
    chartWidth() {
      return (
        this.conWidth *
        this.chart.width /
        (this.chart.width + this.chart.paddingLeft + this.chart.paddingRight)
      );
    },
    // Caculate Chart Height
    chartHeight() {
      return (
        this.conHeight *
        this.chart.height /
        (this.chart.height + this.chart.paddingTop + this.chart.paddingBottom)
      );
    },
    chartTop() {
      return (
        this.conHeight *
        this.chart.paddingTop /
        (this.chart.height + this.chart.paddingTop + this.chart.paddingBottom)
      );
    },
    chartLeft() {
      return (
        this.conWidth *
        this.chart.paddingLeft /
        (this.chart.width + this.chart.paddingLeft + this.chart.paddingRight)
      );
    },
    // X軸線性比例縮放
    xScale() {
      let Xmin = 0;
      let Xmax;
      let newArray = [];

      this.data.value.forEach(function(e) {
        newArray.push(e.number);
      });
      Xmax = d3.max(newArray);

      return d3
        .scaleLinear()
        .domain([Xmin, Xmax])
        .range([0, this.chartWidth]);
    },
    // X軸設定
    xAxis() {
      return d3.axisBottom(this.xScale).tickSizeInner(-this.chartWidth);
    },
  },
  mounted() {
    // 隨機產生資料
    this.randomData();
  },
  methods: {
    initCanvas() {
      let chartContain = document.querySelector(".chartContain");
      let canvas = document.getElementById("canvas");
      let canvasA = document.getElementById("canvasA");

      // Clear Canvas Element
      if (canvas !== null) {
        canvas.parentNode.removeChild(canvas);
        canvasA.parentNode.removeChild(canvasA);
      }

      // Get Container Width
      this.conWidth = chartContain.offsetWidth;

      // For D3 Draw Canvas
      this.canvas = d3
        .select(".chartContain")
        .append("canvas")
        .attr("id", "canvas")
        .attr("class", "chart")
        .attr("width", this.conWidth)
        .attr("height", this.conHeight);
      // For D3 Draw Canvas Animation
      this.canvasA = d3
        .select(".chartContain")
        .append("canvas")
        .attr("id", "canvasA")
        .attr("class", "chartA")
        .attr("width", this.conWidth)
        .attr("height", this.conHeight);

      this.ctx = this.canvas.node().getContext("2d");
      this.ctxA = this.canvasA.node().getContext("2d");

      // Set the parent of all other elements
      this.customBase = document.createElement("custom");
      this.custom = d3.select(this.customBase);

      // Data Bind
      this.dataBind(this.data);
    },
    dataBind(data) {
      /*-------------------------
        X軸
      -------------------------*/
      this.xTick = this.custom
        .append("g")
        .attr("class", "axis axisX")
        .call(this.xAxis)
        .selectAll("g.tick");
      
      // Draw Canvas
      this.drawCanvas();
    },
    drawCanvas() {
      let tickSize = 6; // 軸點大小
      let canvasA = document.querySelector("#canvasA");

      /*-------------------------
        X軸
      -------------------------*/
      // 繪製X軸點
      this.ctx.beginPath();
      // 中間內部灰線
      this.xTick.each((el, index, arr) => {
        let node = d3.select(arr[index]);
        let xTrans = node.attr("transform");
        let xPos = Number(xTrans.split(",")[0].split("(")[1]) + this.chartLeft;

        // 跳過 0 的軸點
        if (index === 0) return;

        // 繪製軸點
        this.ctx.moveTo(xPos, this.chartTop);
        this.ctx.lineTo(xPos, this.chartTop + this.chartHeight);
      });
      this.ctx.setLineDash([4, 6]);
      this.ctx.lineWidth = 2;
      this.ctx.strokeStyle = "#efefef"; // 線顏色
      this.ctx.stroke();

      // 繪製X軸文字
      this.ctx.textAlign = "center";
      this.ctx.textBaseline = "top";
      this.xTick.each((el, index, arr) => {
        let node = d3.select(arr[index]);
        let xTrans = node.attr("transform");
        let xPos = Number(xTrans.split(",")[0].split("(")[1]) + this.chartLeft;

        this.ctx.fillText(
          node.property("innerText"),
          xPos,
          this.chartTop + this.chartHeight
        );
      });

      // 繪製X軸標籤
      this.ctx.textAlign = "center";
      this.ctx.textBaseline = "top";
      this.ctx.font = "16px sans-serif";
      this.ctx.fillText("件數", this.conWidth / 2, this.chartTop + this.chartHeight + 30);

      /*-------------------------
        動畫
      -------------------------*/
      // this.timer = d3.timer(elapsed => {
      //   // Clear Canvas
      //   this.ctxA.clearRect(0, 0, this.conWidth, this.conHeight);
      //   this.animationLine(elapsed);
      // });

      // Canvas On Mouseover
      // canvasA.addEventListener("mousemove", e => {
      //   this.showTooltip(e);
      // });
    },
    randomData() {
      let min = 0;
      let max = 500;
      let month = Math.floor(Math.random() * (12 - 1 + 1)) + 1; // 隨機挑選連續幾個月
      let random = {
        name: "鼓山區",
        value: []
      };

      // 隨機產生資料
      for (let i = 0; i < month; i++) {
        random.value.push({
          month: `${i + 1}月`,
          number: Math.floor(Math.random() * (max - min + 1)) + min
        });
      }

      this.data = random;

      // Init Canvas
      this.initCanvas();

      // Window Resize
      window.addEventListener("resize", this.initCanvas);
    },
  }
};
</script>

<style lang="postcss">
.barpChart {
  /* 動畫 */
  .growBarp-enter-active {
    transition: all 1s;
  }
  .growBarp-enter {
    transform: scaleX(0);
    opacity: 0;
  }
  /* 說明 */
  .detail {
    color: gray;
  }
  /* 統計圖 */
  .chartContain {
    max-width: 600px;
    margin: 0 auto;
    position: relative;
    .chartA {
      position: absolute;
      left: 0;
      top: 0;
      z-index: 1;
    }
    .tooltip {
      min-width: 100px;
      background-color: rgba(0, 0, 0, 0.9);
      color: white;
      padding: 10px;
      border-radius: 6px;
      position: absolute;
      text-align: left;
      line-height: 1.5em;
      z-index: 2;
      &.hidden {
        visibility: hidden;
      }
    }
  }
}
</style>
