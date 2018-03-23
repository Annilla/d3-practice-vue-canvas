<template>
  <div class="lineChart">
    <h1>折線圖</h1>
    <div class="detail">高雄市不動產買賣統計(104年6-10月)</div>
    <!-- 圖表 -->
    <div class="chartContain">
      <svg version="1.1" id="calPath"><path d=""></path></svg>
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
      yTick: undefined, // Y軸 tick
      dots: undefined, // 折點
      lines: undefined, // 折線
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
    // X軸線性比例縮放
    xScale() {
      return d3
        .scaleLinear()
        .domain([0, this.data[0].value.length])
        .range([0, this.chartWidth]);
    },
    // Y軸線性比例縮放
    yScale() {
      let Ymin = 0;
      let Ymax;
      let newArray = [];

      this.data.forEach(function(e, i) {
        e.value.forEach(function(ev) {
          newArray.push(ev.number);
        });
      });
      Ymax = d3.max(newArray);

      return d3
        .scaleLinear()
        .domain([Ymin, Ymax])
        .range([this.chartHeight, 0]);
    },
    // Y軸設定
    yAxis() {
      return d3.axisLeft(this.yScale).tickSizeInner(-this.chart.height);
    },
    // 顏色函數
    color() {
      return d3.scaleOrdinal(d3.schemeCategory10);
    },
    // 資料轉成陣列給之後畫線用
    dataArray() {
      let arrayset = [];

      this.data.forEach(function(e, i) {
        let array = [];

        e.value.forEach(function(ev) {
          array.push(ev.number);
        });

        arrayset.push(array);
      });

      return arrayset;
    },
    // 畫線函數
    line() {
      return d3
        .line()
        .x((d, i) => {
          //利用尺度運算資料索引，傳回x的位置
          return this.xScale(i + 1) + this.chartLeft;
        })
        .y(d => {
          //利用尺度運算資料的值，傳回y的位置
          return this.yScale(d) + this.chartTop;
        });
    },
    // 畫線函數 Canvas
    lineCanvas() {
      return d3
        .line()
        .x((d, i) => {
          //利用尺度運算資料索引，傳回x的位置
          return this.xScale(i + 1) + this.chartLeft;
        })
        .y(d => {
          //利用尺度運算資料的值，傳回y的位置
          return this.yScale(d) + this.chartTop;
        })
        .context(this.ctxA);
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
      let dotIndex = 0; // For dot color
      let nameIndex = 0; // For Name

      /*-------------------------
        Y軸
      -------------------------*/
      this.yTick = this.custom
        .append("g")
        .attr("class", "axis axisY")
        .call(this.yAxis)
        .selectAll("g.tick");

      /*-------------------------
        折線
      -------------------------*/
      // 繪製 line
      this.lines = this.custom
        .selectAll("g.line")
        .data(this.dataArray)
        .enter()
        .append("path")
        .attr("fill", "none")
        .attr("stroke", (d, i) => {
          return this.color(i);
        })
        .attr("d", d => {
          return this.line(d);
        });

      /*-------------------------
        折點
      -------------------------*/
      // 繪製折點 g.dot > circle.circle
      this.dots = this.custom
        .selectAll("g.dot")
        .data(data)
        .enter()
        .append("g") // 塞好 'g'
        .attr("class", "dot") // 準備好 Class
        .selectAll("circle.circle")
        .data(function(d) {
          return d.value;
        })
        .enter()
        .append("circle") // 塞好 'circle'
        .attr("class", "circle"); // 準備好 Class

      // 設置折點 Attribute
      this.dots
        .attr("cx", (d, i) => {
          return this.xScale(i + 1) + this.chartLeft;
        })
        .attr("cy", d => {
          return this.yScale(d.number) + this.chartTop;
        })
        .attr("r", "5")
        .attr("fill", () => {
          dotIndex++;
          return this.color(Math.floor((dotIndex - 1) / 5));
        })
        .attr("name", (d, i) => {
          nameIndex++;
          return this.data[Math.floor((nameIndex - 1) / 5)].name;
        })
        .attr("stroke", "#FFFFFF");

      // Draw Canvas
      this.drawCanvas();
    },
    drawCanvas() {
      let tickSize = 6; // 軸點大小
      let canvasA = document.querySelector("#canvasA");

      // Clear Canvas
      this.ctx.clearRect(0, 0, this.conWidth, this.conHeight);

      /*-------------------------
        X軸
      -------------------------*/
      // 繪製X軸點
      this.ctx.beginPath();
      this.data[0].value.forEach((d, i) => {
        this.ctx.moveTo(
          this.xScale(i + 1) + this.chartLeft,
          this.chartTop + this.chartHeight
        );
        this.ctx.lineTo(
          this.xScale(i + 1) + this.chartLeft,
          this.chartTop + this.chartHeight + tickSize
        );
      });
      this.ctx.stroke();

      // 繪製X軸線
      this.ctx.beginPath();
      this.ctx.moveTo(this.chartLeft, this.chartTop + this.chartHeight);
      this.ctx.lineTo(
        this.chartLeft + this.chartWidth,
        this.chartTop + this.chartHeight
      );
      this.ctx.stroke();

      // 繪製X軸文字
      this.ctx.textAlign = "center";
      this.ctx.textBaseline = "top";
      this.data[0].value.forEach((d, i) => {
        this.ctx.fillText(
          d.month,
          this.xScale(i + 1) + this.chartLeft,
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
      // 繪製Y軸點
      this.ctx.beginPath();
      // 中間內部灰線
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
      this.ctx.strokeStyle = "#e6e6e6"; // 線顏色
      this.ctx.stroke();
      // 最上方的刻點
      this.ctx.beginPath();
      this.ctx.moveTo(this.chartLeft - tickSize, this.chartTop);
      this.ctx.lineTo(this.chartLeft, this.chartTop);
      this.ctx.strokeStyle = "#000"; // 線顏色
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
      let duration = 500;
      let t = Math.min(1, elapsed / duration); // compute how far through the animation we are (0 to 1)

      // Clear Canvas
      this.ctxA.clearRect(0, 0, this.conWidth, this.conHeight);

      /*-------------------------
        折線
      -------------------------*/
      this.lines.each((el, index, arr) => {
        let path = document.querySelector("#calPath path");
        let node = d3.select(arr[index]);
        let totalLength; // Path Length

        path.setAttribute("d", node.attr("d"));
        totalLength = path.getTotalLength();

        this.ctxA.setLineDash([totalLength]);
        this.ctxA.lineDashOffset = totalLength * (1 - t);
        this.ctxA.beginPath(); // 開始繪製
        this.lineCanvas(this.dataArray[index]);
        this.ctxA.strokeStyle = this.color(index); // 線顏色
        this.ctxA.stroke(); // 繪製線
      });

      /*-------------------------
        折點
      -------------------------*/
      this.dots.each((el, index, arr) => {
        let node = d3.select(arr[index]);

        // 開始繪製
        this.ctxA.beginPath();
        // 繪製點
        this.ctxA.arc(
          node.attr("cx"),
          node.attr("cy"),
          Number(node.attr("r")) * t,
          0,
          2 * Math.PI
        );
        // 填色
        this.ctxA.fillStyle = node.attr("fill");
        this.ctxA.fill();
        // 邊框色
        this.ctxA.strokeStyle = node.attr("stroke");
        this.ctxA.stroke();
        // 結束繪製
        this.ctxA.closePath();
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
      let random = [
        {
          name: "鼓山區",
          value: [
            {
              month: "6月",
              number: "233"
            },
            {
              month: "7月",
              number: "412"
            },
            {
              month: "8月",
              number: "533"
            },
            {
              month: "9月",
              number: "267"
            },
            {
              month: "10月",
              number: "321"
            }
          ]
        },
        {
          name: "左營區",
          value: [
            {
              month: "6月",
              number: "432"
            },
            {
              month: "7月",
              number: "443"
            },
            {
              month: "8月",
              number: "334"
            },
            {
              month: "9月",
              number: "233"
            },
            {
              month: "10月",
              number: "114"
            }
          ]
        },
        {
          name: "楠梓區",
          value: [
            {
              month: "6月",
              number: "222"
            },
            {
              month: "7月",
              number: "333"
            },
            {
              month: "8月",
              number: "441"
            },
            {
              month: "9月",
              number: "468"
            },
            {
              month: "10月",
              number: "378"
            }
          ]
        }
      ];

      // 隨機產生資料
      for (let i = 0; i < 3; i++) {
        random[i].value.forEach(e => {
          e.number = Math.floor(Math.random() * (max - min + 1)) + min;
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
      let pointCircle = false;

      this.dots.each((el, index, arr) => {
        let node = d3.select(arr[index]);

        // 開始繪製
        this.ctxA.beginPath();
        // 點 Path
        this.ctxA.arc(
          node.attr("cx"),
          node.attr("cy"),
          node.attr("r"),
          0,
          2 * Math.PI
        );
        // 如果滑過點
        if (this.ctxA.isPointInPath(x, y) && !pointCircle) {
          // 設置位置
          document
            .querySelector(".tooltip")
            .setAttribute("style", `left: ${mouseX}px; top: ${mouseY}px;`);
          // 插入名稱
          document.querySelector(".tooltip .name").innerHTML = `${node.attr(
            "name"
          )} / ${el.month}`;
          document.querySelector(".tooltip .value").innerHTML = `${
            el.number
          } 件`;

          // Tooltip 區塊
          this.hideTooltip = false;
          pointCircle = true;
        } else if (!this.ctxA.isPointInPath(x, y) && !pointCircle) {
          this.hideTooltip = true;
        }
      });
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
    position: relative;
    #calPath {
      display: none;
    }
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
