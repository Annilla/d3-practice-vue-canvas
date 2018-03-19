<template>
  <div class="lineChart">
    <h1>折線圖</h1>
    <div class="detail">高雄市不動產買賣統計(104年6-10月)</div>
    <!-- 圖表 -->
    <div class="chartContain">
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
      conWidth: 0, // Get Container Width
      canvas: undefined, // For D3 Draw Canvas
      ctx: undefined, // Set Canvas 2D
      customBase: undefined, // This is the parent of all other elements
      custom: undefined,
      dots: undefined, // 折點
      lines: undefined // 折線
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
    conHeight() {
      return this.conWidth * this.conRate;
    },
    chartWidth() {
      return (
        this.conWidth *
        this.chart.width /
        (this.chart.width + this.chart.paddingLeft + this.chart.paddingRight)
      );
    },
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
        })
        .context(this.ctx);
    }
  },
  mounted() {
    // 隨機產生資料
    this.randomData();

    // Init Canvas
    this.initCanvas();

    // Window Resize
    window.addEventListener("resize", this.initCanvas);
  },
  methods: {
    initCanvas() {
      let chartContain = document.querySelector(".chartContain");

      // Clear Container Element
      chartContain.innerHTML = "";

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

      // Data Bind
      this.dataBind(this.data);
    },
    dataBind(data) {
      let dotIndex = 0; // For dot color

      /*-------------------------
        折線
      -------------------------*/

      /*-------------------------
        折點
      -------------------------*/
      // 繪製折點 g.dot > circle.circle
      // this.dots = this.custom
      //   .selectAll("g.dot")
      //   .data(data)
      //   .enter()
      //   .append("g") // 塞好 'g'
      //   .attr("class", "dot") // 準備好 Class
      //   .selectAll("circle.circle")
      //   .data(function(d) {
      //     return d.value;
      //   })
      //   .enter()
      //   .append("circle") // 塞好 'circle'
      //   .attr("class", "circle"); // 準備好 Class

      // 設置折點 Attribute
      // this.dots
      //   .attr("cx", (d, i) => {
      //     return this.xScale(i + 1);
      //   })
      //   .attr("cy", d => {
      //     return this.yScale(d.number);
      //   })
      //   .attr("r", "5")
      //   .attr("fill", () => {
      //     dotIndex++;
      //     return this.color(Math.floor((dotIndex - 1) / 5));
      //   })
      //   .attr("stroke", "#FFFFFF");

      // Draw Canvas
      this.drawCanvas();
    },
    drawCanvas() {
      let tickSize = 6; // X軸點大小
      let tickYCount = 10; // Y軸點數
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

      /*-------------------------
        Y軸
      -------------------------*/
      // 繪製Y軸點
      this.ctx.beginPath();
      for (let i = 0; i < tickYCount; i++) {
        this.ctx.moveTo(
          this.chartLeft - tickSize,
          this.chartTop + this.chartHeight / tickYCount * i
        );
        this.ctx.lineTo(
          this.chartLeft,
          this.chartTop + this.chartHeight / tickYCount * i
        );
      }
      this.ctx.stroke();

      // 繪製Y軸線
      this.ctx.beginPath();
      this.ctx.moveTo(this.chartLeft, this.chartTop);
      this.ctx.lineTo(this.chartLeft, this.chartTop + this.chartHeight);
      this.ctx.stroke();

      // 繪製Y軸文字
      // this.ctx.textAlign = "center";
      // this.ctx.textBaseline = "top";
      // for (let i = 0; i < tickYCount; i++) {
      //   this.ctx.fillText(
      //     i,
      //     -tickSize,
      //     this.chartTop + this.chartHeight / tickYCount * i
      //   );
      // }

      /*-------------------------
        折線
      -------------------------*/
      this.dataArray.forEach((el, index) => {
        this.ctx.beginPath(); // 開始繪製
        this.line(el); // 使用 D3 line 函數
        this.ctx.strokeStyle = this.color(index); // 線顏色
        this.ctx.stroke(); // 繪製線
      });

      /*-------------------------
        折點
      -------------------------*/
      // this.dots.each((value, index, el) => {
      //   let node = d3.select(el[index]);

      //   // 開始繪製
      //   this.ctx.beginPath();
      //   // 繪製點
      //   this.ctx.arc(
      //     node.attr("cx"),
      //     node.attr("cy"),
      //     node.attr("r"),
      //     0,
      //     2 * Math.PI
      //   );
      //   // 填色
      //   this.ctx.fillStyle = node.attr("fill");
      //   this.ctx.fill();
      //   // 邊框色
      //   this.ctx.strokeStyle = node.attr("stroke");
      //   this.ctx.stroke();
      //   // 結束繪製
      //   this.ctx.closePath();
      // });
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
      // for (let i = 0; i < 3; i++) {
      //   random[i].value.forEach(e => {
      //     e.number = Math.floor(Math.random() * (max - min + 1)) + min;
      //   });
      // }

      this.data = random;
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
  }
}
</style>
