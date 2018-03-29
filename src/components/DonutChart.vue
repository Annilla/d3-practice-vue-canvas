<template>
  <div class="donutChart">
    <h1>甜甜圈圖</h1>
    <div class="detail">高雄市不動產買賣統計(104年6月)</div>
    <!-- 圖表 -->
    <div class="chartContain">
      <div class="chartWrap"></div>
      <ul class="labelWrap">
        <li
          v-for="(p, key) in pie"
          :key="`${key}${p.d}`">
          <span class="color" :style="`background-color: ${p.color};`"></span>
          <span class="region">{{ data[key].name }}</span>
          <span class="percentage">{{ p.percentage }}</span>
        </li>
      </ul>
      <div :class="{ tooltip: true, hidden: hideTooltip}">
        <div class="name">左營區 / 10%</div>
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
        innerRadius: 100, // 0 for PieChart
        outerRadius: 200
      },
      hideTooltip: true,
      conWidth: 0, // Get Container Width
      canvas: undefined, // For D3 Draw Canvas
      ctx: undefined, // Set Canvas 2D
      customBase: undefined, // This is the parent of all other elements
      custom: undefined,
      timer: undefined // For Animation Timer
    };
  },
  computed: {
    // Caculate Container Height 正圓長寬 1:1
    conHeight() {
      return this.conWidth;
    },
    // Caculate Chart OuterRadius
    chartOuterRadius() {
      return this.conWidth / 2;
    },
    // Caculate Chart InnerRadius
    chartInnerRadius() {
      return this.chartOuterRadius * this.chart.innerRadius / this.chart.outerRadius;
    },
    // Circle 半徑
    radius() {
      return (
        this.chartOuterRadius -
        (this.chartOuterRadius - this.chartInnerRadius) / 2
      );
    },
    // 顏色函數
    color() {
      return d3.scaleOrdinal(d3.schemeCategory20c);
    },
    totalSum() {
      let sum = 0;

      if (this.data.length) {
        this.data.forEach((e, i) => {
          sum = sum + e.value;
        });
      }

      return sum;
    },
    pie() {
      let newArray = [];
      let format = d3.format(".0%"); // 百分比單位
      let pie = d3
        .pie()
        .sort(null)
        .value(function(d) {
          return d.value;
        })(this.data); // 準備好 arc 角度

      if (this.data.length) {
        this.data.forEach((e, i) => {
          // 新增陣列
          newArray.push({
            startAngle: pie[i].startAngle - Math.PI / 2,
            endAngle: pie[i].endAngle - Math.PI / 2,
            percentage: format(e.value / this.totalSum),
            color: this.color(i),
            name: e.name,
            value: e.value
          });
        });
      }

      return newArray;
    }
  },
  mounted() {
    // 隨機產生資料
    this.randomData();
  },
  methods: {
    initCanvas() {
      let chartContain = document.querySelector(".chartWrap");
      let canvas = document.getElementById("canvas");

      // Clear Canvas Element
      if (canvas !== null) {
        canvas.parentNode.removeChild(canvas);
      }

      // Get Container Width
      this.conWidth = chartContain.offsetWidth;

      // For D3 Draw Canvas
      this.canvas = d3
        .select(".chartWrap")
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
      let duration = 700;
      let t = Math.min(1, elapsed / duration); // compute how far through the animation we are (0 to 1)

      // Clear Canvas
      this.ctx.clearRect(0, 0, this.conWidth, this.conHeight);

      /*-------------------------
        甜甜圈
      -------------------------*/
      this.ctx.save();
      this.pie.forEach((e, i) => {
        // 開始繪製
        this.ctx.beginPath();
        this.ctx.arc(
          this.conWidth / 2,
          this.conHeight / 2,
          this.radius,
          e.startAngle,
          e.startAngle + (e.endAngle - e.startAngle) * t
        );
        // 邊框色
        this.ctx.lineWidth = this.chartOuterRadius - this.chartInnerRadius;
        this.ctx.strokeStyle = e.color;
        this.ctx.globalAlpha = t;
        this.ctx.stroke();
      });

      /*-------------------------
        甜甜圈文字
      -------------------------*/
      this.pie.forEach((e, i) => {
        let theta = e.startAngle + (e.endAngle - e.startAngle) / 2;
        let x = this.radius * Math.cos(theta);
        let y = this.radius * Math.sin(theta);

        // 開始繪製
        this.ctx.textAlign = "center";
        this.ctx.textBaseline = "middle";
        this.ctx.font = "16px sans-serif";
        this.ctx.fillStyle = "#fff";
        this.ctx.fillText(
          e.percentage,
          this.conWidth / 2 + x,
          this.conHeight / 2 + y
        );
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
      let random = [
        {
          name: "鼓山區",
          value: 233
        },
        {
          name: "左營區",
          value: 290
        },
        {
          name: "楠梓區",
          value: 355
        },
        {
          name: "三民區",
          value: 340
        },
        {
          name: "苓雅區",
          value: 173
        },
        {
          name: "前鎮區",
          value: 199
        },
        {
          name: "小港區",
          value: 145
        },
        {
          name: "鳳山區",
          value: 394
        },
        {
          name: "大寮區",
          value: 151
        },
        {
          name: "仁武區",
          value: 139
        },
        {
          name: "岡山區",
          value: 107
        }
      ];

      // 隨機砍掉區域
      let newRandom = [];
      random.forEach((el, index) => {
        if (Math.random() >= 0.5) {
          newRandom.push(el);
        }
      });

      // 隨機產生資料
      newRandom.forEach(el => {
        el.value = Math.floor(Math.random() * (max - min + 1)) + min;
      });

      this.data = newRandom;

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
      let pointCircle = false;

      this.pie.forEach((e, i) => {
        this.ctx.beginPath();
        // 外圓弧
        this.ctx.arc(
          this.conWidth / 2,
          this.conHeight / 2,
          this.chartOuterRadius,
          e.startAngle,
          e.endAngle
        );
        // 內圓弧
        this.ctx.arc(
          this.conWidth / 2,
          this.conHeight / 2,
          this.chartInnerRadius,
          e.endAngle,
          e.startAngle,
          true // 反時鐘連接
        );
        this.ctx.closePath();
        // 如果滑過點
        if (this.ctx.isPointInPath(x, y) && !pointCircle) {
          // 設置位置
          document
            .querySelector(".tooltip")
            .setAttribute("style", `left: ${mouseX}px; top: ${mouseY}px;`);
          // 插入名稱
          document.querySelector(".tooltip .name").innerHTML = `${
            e.name
          } / 6月`;
          document.querySelector(".tooltip .value").innerHTML = `${
            e.value
          } 件`;

          // Tooltip 區塊
          this.hideTooltip = false;
          pointCircle = true;
        } else if (!this.ctx.isPointInPath(x, y) && !pointCircle) {
          this.hideTooltip = true;
        }
      });
    }
  }
};
</script>

<style lang="postcss">
.donutChart {
  /* 說明 */
  .detail {
    color: gray;
  }
  /* 統計圖 */
  .chartContain {
    max-width: 600px;
    margin: 15px auto;
    overflow: hidden;
    position: relative;
    .chartWrap {
      width: 100%;
      position: relative;
      @media screen and (min-width: 600px) {
        width: calc(100% - 200px);
        float: left;
      }
    }
    ul.labelWrap {
      width: 100%;
      padding: 0;
      overflow: hidden;
      @media screen and (min-width: 600px) {
        width: 150px;
        padding-left: 50px;
        float: right;
      }
      li {
        list-style: none;
        display: block;
        overflow: hidden;
        margin: 15px 10px 0 0;
        float: left;
        .color {
          display: inline-block;
          width: 10px;
          height: 10px;
          margin: 5px 5px 0 0;
          float: left;
        }
        .region {
          float: left;
          margin-right: 10px;
        }
        .percentage {
          float: left;
          color: rgba(0, 0, 0, 0.4);
        }
      }
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
