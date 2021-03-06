<template>
  <div class="barVChart">
    <h1>直條圖</h1>
    <div class="detail">高雄市不動產買賣統計(104年)</div>
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
        paddingLeft: 60,
        gap: 40
      },
      hideTooltip: true,
      conWidth: 0, // Get Container Width
      canvas: undefined, // For D3 Draw Canvas
      ctx: undefined, // Set Canvas 2D
      customBase: undefined, // This is the parent of all other elements
      custom: undefined,
      xTick: undefined, // X軸 tick
      yTick: undefined, // Y軸 tick
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
    chartGap() {
      return (
        this.conWidth *
        this.chart.gap /
        (this.chart.width + this.chart.paddingLeft + this.chart.paddingRight)
      );
    },
    valueLength() {
      return this.data[0] ? this.data[0].value.length : 0;
    },
    // X軸設定
    xAxis() {
      let tickNum =
        this.valueLength < 3 ? this.valueLength + 1 : this.valueLength;

      return d3
        .axisBottom(this.xScale)
        .ticks(tickNum)
        .tickFormat((d, i) => {
          return this.xLabel[i];
        });
    },
    // Y軸設定
    yAxis() {
      return d3.axisLeft(this.yScale).tickSizeInner(-this.chartHeight);
    },
    // X軸線性比例縮放
    xScale() {
      return d3
        .scaleLinear()
        .domain([0, this.data[0].value.length + 1])
        .range([0, this.chartWidth]);
    },
    // Y軸線性比例縮放
    yScale() {
      let Ymin = 0;
      let Ymax;
      let newArray = [];

      this.data.forEach(group => {
        group.value.forEach(el => {
          newArray.push(el.number);
        });
      });
      Ymax = d3.max(newArray);

      return d3
        .scaleLinear()
        .domain([Ymin, Ymax])
        .range([this.chartHeight, 0]);
    },
    // X軸標籤文字
    xLabel() {
      let tickLabels = [""];

      this.data[0].value.forEach(function(e) {
        tickLabels.push(e.month);
      });

      return tickLabels;
    },
    // Y軸文字座標
    axisYText() {
      let x = 0 - this.chartHeight / 2;
      let y = 0 - this.chartLeft;
      return [x, y];
    },
    // 顏色函數
    color() {
      return d3.scaleOrdinal(d3.schemeCategory20c);
    },
    barWidth() {
      /* --------------------
      1. 把圖表寬度扣掉左右不放圖的間距
         this.chartWidth / (this.valueLength + 1) * this.valueLength
      2. 扣掉長條圖中間的間隔
         - (this.chartGap * this.data.length)
      3. 剩下的寬度分給所有長條
         / (this.valueLength * this.data.length)
      -------------------- */
      return (
        (this.chartWidth / (this.valueLength + 1) * this.valueLength -
          this.chartGap * this.data.length) /
        (this.valueLength * this.data.length)
      );
    },
    bars() {
      // 直條
      let barArray = [];

      this.data.forEach((d, index) => {
        d.value.forEach((e, i) => {
          barArray.push({
            color: this.color(index),
            /* --------------------
            1. 把群組直條往左推整體的一半
                - this.barWidth * this.data.length / 2
            2. 依個別順序向右平移
                + this.barWidth * index
            -------------------- */
            x:
              this.chartLeft +
              this.xScale(i + 1) -
              this.barWidth * this.data.length / 2 +
              this.barWidth * index,
            y: this.chartTop + this.yScale(e.number),
            width: this.barWidth,
            height: this.chartHeight - this.yScale(e.number),
            name: this.data[index].name,
            month: e.month,
            number: e.number
          });
        });
      });

      return barArray;
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

      // Clear Canvas Element
      if (canvas !== null) {
        canvas.parentNode.removeChild(canvas);
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

      this.ctx = this.canvas.node().getContext("2d");

      // Set the parent of all other elements
      this.customBase = document.createElement("custom");
      this.custom = d3.select(this.customBase);

      // Draw Canvas
      this.drawCanvas();
    },
    drawStatic() {
      let tickSize = 6; // 軸點大小

      /*-------------------------
        X軸
      -------------------------*/
      this.xTick = this.custom
        .append("g")
        .attr("class", "axis axisX")
        .call(this.xAxis)
        .selectAll("g.tick");
      // 繪製X軸點
      this.ctx.beginPath();
      // 黑色刻點
      this.xTick.each((el, index, arr) => {
        let node = d3.select(arr[index]);
        let xTrans = node.attr("transform");
        let xPos = Number(xTrans.split(",")[0].split("(")[1]) + this.chartLeft;

        // 跳過 0 的軸點
        if (index === 0) return;

        // 繪製軸點
        this.ctx.moveTo(xPos, this.chartTop + this.chartHeight);
        this.ctx.lineTo(xPos, this.chartTop + this.chartHeight + tickSize);
      });
      this.ctx.stroke();

      // 繪製X軸線
      this.ctx.beginPath();
      this.ctx.moveTo(this.chartLeft, this.chartTop + this.chartHeight);
      this.ctx.lineTo(
        this.chartLeft + this.chartHeight,
        this.chartTop + this.chartWidth
      );
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
          this.chartTop + this.chartHeight + tickSize
        );
      });

      // 繪製X軸標籤
      this.data.forEach((el, index) => {
        // 圓點
        // 開始繪製
        this.ctx.save();
        this.ctx.beginPath();
        // 繪製點
        this.ctx.arc(
          this.chartLeft + index * 100,
          this.chartTop + this.chartHeight + 50,
          5,
          0,
          2 * Math.PI
        );
        // 填色
        this.ctx.fillStyle = this.color(index);
        this.ctx.fill();
        // 結束繪製
        this.ctx.closePath();

        // 文字
        this.ctx.textAlign = "left";
        this.ctx.textBaseline = "middle";
        this.ctx.font = "16px sans-serif";
        this.ctx.fillStyle = "#000";
        this.ctx.fillText(
          el.name,
          this.chartLeft + index * 100 + 10,
          this.chartTop + this.chartHeight + 50
        );
        this.ctx.restore();
      });

      /*-------------------------
        Y軸
      -------------------------*/
      this.yTick = this.custom
        .append("g")
        .attr("class", "axis axisY")
        .call(this.yAxis)
        .selectAll("g.tick");
      // 繪製Y軸點
      this.ctx.save();
      this.ctx.beginPath();
      // 內部灰色刻線
      this.yTick.each((el, index, arr) => {
        let node = d3.select(arr[index]);
        let yTrans = node.attr("transform");
        let yPos = Number(yTrans.split(",")[1].split(")")[0]) + this.chartTop;

        // 跳過 0 的軸點
        if (index === 0) return;

        // 繪製軸點
        this.ctx.moveTo(this.chartLeft, yPos);
        this.ctx.lineTo(this.chartLeft + this.chartWidth, yPos);
      });
      this.ctx.strokeStyle = "#efefef"; // 線顏色
      this.ctx.stroke();
      this.ctx.restore();

      // 最上方的刻點
      this.ctx.beginPath();
      this.ctx.moveTo(this.chartLeft - tickSize, this.chartTop);
      this.ctx.lineTo(this.chartLeft, this.chartTop);
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

      // 繪製Y軸標籤
      // start by saving the current context (current orientation, origin)
      this.ctx.save();
      this.ctx.translate(0, 0);
      this.ctx.rotate(-Math.PI / 2);
      this.ctx.textAlign = "center";
      this.ctx.textBaseline = "top";
      this.ctx.font = "16px sans-serif";
      this.ctx.fillText("件數", -(this.chartTop + this.chartHeight / 2), 0);
      this.ctx.restore(); // now restore the canvas flipping it back to its original orientation
    },
    drawCanvas() {
      let canvas = document.querySelector("#canvas");

      /*-------------------------
        動畫
      -------------------------*/
      this.timer = d3.timer(elapsed => {
        this.animationLine(elapsed);
      });

      // Canvas On Mouseover
      canvas.addEventListener("mousemove", e => {
        this.showTooltip(e);
      });
    },
    animationLine(elapsed) {
      let duration = 800;
      let t = Math.min(1, elapsed / duration); // compute how far through the animation we are (0 to 1)

      // Clear Canvas
      this.ctx.clearRect(0, 0, this.conWidth, this.conHeight);

      // 繪製不須動畫的元素
      this.drawStatic();

      /*-------------------------
        直條
      -------------------------*/
      this.ctx.save();
      this.bars.forEach((e, i) => {
        // 開始繪製
        this.ctx.beginPath();
        this.ctx.rect(e.x, e.y + e.height * (1 - t), e.width, e.height * t);
        this.ctx.globalAlpha = t;
        this.ctx.fillStyle = e.color;
        this.ctx.fill();
      });

      /*-------------------------
        直條文字
      -------------------------*/
      this.ctx.font = "12px sans-serif";
      this.ctx.textBaseline = "bottom";
      this.ctx.textAlign = "center";
      this.ctx.fillStyle = "gray";
      this.bars.forEach((e, i) => {
        // 開始繪製
        this.ctx.fillText(e.number, e.x + this.barWidth / 2, e.y);
      });
      this.ctx.restore();

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
      let random = [
        {
          name: "鼓山區",
          value: []
        },
        {
          name: "左營區",
          value: []
        },
        {
          name: "楠梓區",
          value: []
        }
      ];

      // 隨機產生資料
      random.forEach(el => {
        for (let i = 0; i < month; i++) {
          el.value.push({
            month: `${i + 1}月`,
            number: Math.floor(Math.random() * (max - min + 1)) + min
          });
        }
      });

      this.data = random;

      // Init Canvas
      this.initCanvas();

      // Window Resize
      window.addEventListener("resize", this.initCanvas);
    },
    showTooltip(e) {
      // Correct mouse position:
      let canvas = document.querySelector("#canvas");
      let rect = canvas.getBoundingClientRect();
      let x = e.clientX - rect.left;
      let y = e.clientY - rect.top;
      let mouseX = x + 20;
      let mouseY = y;
      let pointRec = false;

      this.bars.forEach((e, i) => {
        // 開始繪製
        this.ctx.beginPath();
        this.ctx.rect(e.x, e.y, e.width, e.height);
        // 如果滑過
        if (this.ctx.isPointInPath(x, y) && !pointRec) {
          // 設置位置
          document
            .querySelector(".tooltip")
            .setAttribute("style", `left: ${mouseX}px; top: ${mouseY}px;`);
          // 插入名稱
          document.querySelector(".tooltip .name").innerHTML = `${e.name} / ${
            e.month
          }`;
          document.querySelector(".tooltip .value").innerHTML = `${
            e.number
          } 件`;

          // Tooltip 區塊
          this.hideTooltip = false;
          pointRec = true;
        } else if (!this.ctx.isPointInPath(x, y) && !pointRec) {
          this.hideTooltip = true;
        }
      });
    }
  }
};
</script>

<style lang="postcss">
.barVChart {
  /* 說明 */
  .detail {
    color: gray;
  }
  /* 統計圖 */
  .chartContain {
    max-width: 600px;
    margin: 0 auto;
    position: relative;
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
