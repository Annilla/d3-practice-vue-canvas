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
      yTick: undefined, // Y軸 tick
      bars: undefined, // 繪製橫條
      timer: undefined // For Animation Timer
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
    valueLength() {
      return this.data.value ? this.data.value.length : 0;
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
    // Y軸線性比例縮放
    yScale() {
      return d3
        .scaleLinear()
        .domain([0, this.data.value.length + 1])
        .range([this.chartHeight, 0]);
    },
    // X軸設定
    xAxis() {
      return d3.axisBottom(this.xScale).tickSizeInner(-this.chartWidth);
    },
    // Y軸設定
    yAxis() {
      let tickNum =
        this.valueLength < 3 ? this.valueLength + 1 : this.valueLength;

      return d3
        .axisLeft(this.yScale)
        .ticks(tickNum)
        .tickFormat((d, i) => {
          return this.yLabel[i];
        });
    },
    // Y軸標籤文字
    yLabel() {
      let tickLabels = [""];

      this.data.value.forEach(function(e) {
        tickLabels.push(e.month);
      });

      return tickLabels;
    },
    // 顏色函數
    color() {
      return d3.scaleOrdinal(d3.schemeCategory10);
    },
    barWidth() {
      let gap = 20;
      return this.chartHeight / (this.valueLength + 1) - gap;
    }
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

      /*-------------------------
        Y軸
      -------------------------*/
      this.yTick = this.custom
        .append("g")
        .attr("class", "axis axisY")
        .call(this.yAxis)
        .selectAll("g.tick");

      /*-------------------------
        橫條
      -------------------------*/
      // 繪製 bar
      this.bars = this.custom
        .append("g")
        .attr("class", "bars")
        .selectAll("rect.bar")
        .data(data.value)
        .enter()
        .append("rect") //塞好'rect'
        .attr("class", "bar"); //準備好Class

      // 設置橫條 Attribute
      this.bars
        .attr("fill", this.color(0))
        .attr("x", this.chartLeft)
        .attr("y", (d, i) => {
          return this.chartTop + this.yScale(i + 1) - this.barWidth / 2;
        })
        .attr("width", d => {
          return this.xScale(d.number);
        })
        .attr("height", this.barWidth);

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
      this.ctx.save();
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
      this.ctx.restore();

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
      this.ctx.save();
      this.ctx.textAlign = "center";
      this.ctx.textBaseline = "top";
      this.ctx.font = "16px sans-serif";
      this.ctx.fillText(
        "件數",
        this.conWidth / 2,
        this.chartTop + this.chartHeight + 30
      );
      this.ctx.restore();

      /*-------------------------
        Y軸
      -------------------------*/
      // 繪製Y軸點
      this.ctx.beginPath();
      // 黑色刻點
      this.yTick.each((el, index, arr) => {
        let node = d3.select(arr[index]);
        let yTrans = node.attr("transform");
        let yPos = Number(yTrans.split(",")[1].split(")")[0]) + this.chartTop;

        // 跳過 0 的軸點
        if (index === 0) return;

        // 繪製軸點
        this.ctx.moveTo(this.chartLeft - tickSize, yPos);
        this.ctx.lineTo(this.chartLeft, yPos);
      });
      this.ctx.stroke();
      // 最下方的刻點
      this.ctx.beginPath();
      this.ctx.moveTo(
        this.chartLeft - tickSize,
        this.chartTop + this.chartHeight
      );
      this.ctx.lineTo(this.chartLeft, this.chartTop + this.chartHeight);
      this.ctx.stroke();

      // 繪製Y軸線
      this.ctx.beginPath();
      this.ctx.moveTo(this.chartLeft, this.chartTop);
      this.ctx.lineTo(this.chartLeft, this.chartTop + this.chartHeight);
      this.ctx.stroke();

      // 繪製Y軸文字
      this.ctx.textAlign = "right";
      this.ctx.textBaseline = "middle";
      this.yTick.each((el, index, arr) => {
        let node = d3.select(arr[index]);
        let yTrans = node.attr("transform");
        let yPos = Number(yTrans.split(",")[1].split(")")[0]) + this.chartTop;

        this.ctx.fillText(
          node.property("innerText"),
          this.chartLeft - tickSize,
          yPos
        );
      });

      /*-------------------------
        動畫
      -------------------------*/
      this.timer = d3.timer(elapsed => {
        this.animationLine(elapsed);
      });

      // Canvas On Mouseover
      canvasA.addEventListener("mousemove", e => {
        this.showTooltip(e);
      });
    },
    animationLine(elapsed) {
      let duration = 800;
      let t = Math.min(1, elapsed / duration); // compute how far through the animation we are (0 to 1)

      // Clear Canvas
      this.ctxA.clearRect(0, 0, this.conWidth, this.conHeight);

      /*-------------------------
        橫條
      -------------------------*/
      this.bars.each((el, index, arr) => {
        let node = d3.select(arr[index]);

        // 開始繪製
        this.ctxA.beginPath();
        this.ctxA.rect(
          node.attr("x"),
          node.attr("y"),
          Number(node.attr("width")) * t,
          node.attr("height")
        );
        this.ctxA.globalAlpha = t;
        this.ctxA.fillStyle = node.attr("fill");
        this.ctxA.fill();
      });

      /*-------------------------
        橫條文字
      -------------------------*/
      this.ctxA.textBaseline = "middle";
      this.ctxA.font = "16px sans-serif";
      this.bars.each((el, index, arr) => {
        let node = d3.select(arr[index]);
        let x = Number(node.attr("width")) + this.chartLeft - 5; // 修正x太低數值的文字改在橫條外顯示

        // 修正x太低數值的文字改在橫條外顯示
        if(x < this.chartLeft + 50) {
          this.ctxA.fillStyle = "#000";
          this.ctxA.textAlign = "left";
          x = Number(node.attr("width")) + this.chartLeft + 5;
        } else {
          this.ctxA.fillStyle = "#fff";
          this.ctxA.textAlign = "right";
        }

        this.ctxA.fillText(
          el.number,
          x,
          Number(node.attr("y")) + this.barWidth / 2
        );
        
      });

      // if this animation is over
      if (t === 1) {
        // stop this timer since we are done animating.
        this.timer.stop();
      }
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
    showTooltip(e) {
      // Correct mouse position:
      let canvas = document.querySelector("#canvasA");
      let rect = canvas.getBoundingClientRect();
      let x = e.clientX - rect.left;
      let y = e.clientY - rect.top;
      let mouseX = x + 20;
      let mouseY = y;
      let pointRec = false;
      
      this.bars.each((el, index, arr) => {
        let node = d3.select(arr[index]);

        // 開始繪製
        this.ctxA.beginPath();
        this.ctxA.rect(
          node.attr("x"),
          node.attr("y"),
          node.attr("width"),
          node.attr("height")
        );
        // 如果滑過
        if (this.ctxA.isPointInPath(x, y) && !pointRec) {
          // 設置位置
          document
            .querySelector(".tooltip")
            .setAttribute("style", `left: ${mouseX}px; top: ${mouseY}px;`);
          // 插入名稱
          document.querySelector(".tooltip .name").innerHTML = `${this.data.name} / ${el.month}`;
          document.querySelector(".tooltip .value").innerHTML = `${
            el.number
          } 件`;

          // Tooltip 區塊
          this.hideTooltip = false;
          pointRec = true;
        } else if (!this.ctxA.isPointInPath(x, y) && !pointRec) {
          this.hideTooltip = true;
        }
      });
    }
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
