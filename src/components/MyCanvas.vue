<template>
  <div class="wrapper">
    <canvas
      :style="{width: maxValue + 'px', height: maxValue + 'px'}"
      id="my-canvas"
      ref="myCanvas"
    />
    <div class="add-buttons">
      <div class="input-data">
        <button @click="setInput">Задать входные данные</button>
      </div>
      <div class="inputs">
        <div
          v-for="(n, index) in inputsCount"
          :key="'div' + index"
          class="item"
        >
          <label :key="'label' + n">
            {{ index === 0 || index % 2 === 0 ? 'x' : 'y' }}
            <input
              :key="'input' + n"
              v-model.number="points[index]"
              type="number"
              min="0"
            >
          </label>
        </div>
      </div>
      <div class="add-point" v-if="inputsCount > 0">
        <button v-if="showAddPointButton" @click="addPoint">Добавить точку</button>
        <button v-if="!showAddPointButton" @click="handlePoints">Обработать</button>
        <button style="margin-left:10px;" @click="clearInputs">Очистить</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      canvas: null,
      ctx: null,
      incomePoints: [],
      points: [],
      maxValue: null,
      showInputs: false,
      inputsCount: 0,
      showAddPointButton: true,
    };
  },
  mounted() {
    fetch('/json/input.json').then(response => response.json()).then((response) => {
      this.incomePoints = response.items;
      this.getPoints(this.incomePoints);
      this.getMaxValue(this.points);
      return;
    }).then(() => {
      this.canvas = this.$refs.myCanvas;
      this.canvas.width = this.maxValue;
      this.canvas.height = this.maxValue;
      this.ctx = this.canvas.getContext('2d');
      this.ctx.font = '14px sans-serif';

      this.makeLines(this.points);
    });
  },
  methods: {
    getPoints(incomeArray) {
      const len = incomeArray.length;
      for (let i = 0; i < len; i++) {
        const result = incomeArray[i].split(';');
        this.points.push(...result);
      }
      this.points = this.points.map(el => +el);
    },
    getMaxValue(points) {
      this.maxValue = Math.max(...points);
      this.maxValue += 20;
    },
    checkIntersect() {
      if (this.points.length < 8) {
        this.showAddPointButton = true;
        alert('Для построения отрезков необходимо задать 4 точки')
      }
      if (this.lineSegmentsIntersect(...this.points)) {
        this.movePoint();
      }
    },
    lineSegmentsIntersect(x1, y1, x2, y2, x3, y3, x4, y4) {
      let a_dx = x2 - x1;
      let a_dy = y2 - y1;
      let b_dx = x4 - x3;
      let b_dy = y4 - y3;
      let s = (-a_dy * (x1 - x3) + a_dx * (y1 - y3)) / (-b_dx * a_dy + a_dx * b_dy);
      let t = (+b_dx * (y1 - y3) - b_dy * (x1 - x3)) / (-b_dx * a_dy + a_dx * b_dy);
      return (s >= 0 && s <= 1 && t >= 0 && t <= 1) ? [x1 + t * a_dx, y1 + t * a_dy] : false;
    },
    makeLines(points) {
      this.ctx.beginPath();
      this.ctx.moveTo(points[0], points[1]);
      this.ctx.fillText(`1 (${points[0]}, ${points[1]})`, points[0], points[1]);
      for (let i = 2; i < points.length; i += 2) {
        this.ctx.lineTo(points[i], points[i + 1]);
        this.ctx.fillText(`${i / 2 + 1} (${points[i]}, ${points[i + 1]})`, points[i], points[i + 1]);
      }
      this.ctx.stroke();
    },
    setInput() {
      this.showInputs = true;
      this.points = [];
      this.inputsCount = 2;
      this.showAddPointButton = true;
    },
    addPoint() {
      if (this.inputsCount === 8) {
        this.showAddPointButton = false;
        return;
      }
      this.inputsCount += 2;
    },
    rewriteCanvas() {
      this.ctx.clearRect(0, 0, this.maxValue, this.maxValue)
      this.makeLines(this.points);
    },
    movePoint() {
      const moved = this.points.splice(this.points.length - 1, 2);
      this.points.unshift(...moved);
      this.checkIntersect();
      this.rewriteCanvas();
    },
    handlePoints() {
      this.rewriteCanvas();
      this.checkIntersect();
    },
    clearInputs() {
      this.points = [];
      this.inputsCount = 0;
      this.showAddPointButton = true;
    }
  }
};
</script>

<style scoped lang="scss">
.input-data {
  width: 200px;
  margin: 0 auto;
  button {
    width: 100%;
  }
}
.inputs {
  width: 300px;
  margin: 0 auto;
  margin-top: 15px;
  .item {
    &:nth-child(n+2) {
      margin-top: 10px;
    }
  }
}
.add-point {
  margin-top: 15px;
}
button {
  border: none;
  outline: none;
  color: #fff;
  padding: 10px 20px;
  text-align: center;
  font-size: 14px;
  cursor: pointer;
  background: #927AFF;
  box-shadow: 0px 10px 20px rgba(189, 155, 238, 0.38), 0px 6px 8px rgba(0, 0, 0, 0.06);
  border-radius: 50px;
}
input {
  background: #FDFDFD;
  border: 1px solid #f5f5f5;
  box-sizing: border-box;
  height: 40px;
  outline: none;
  padding-left: 20px;
  background: #FFFFFF;
  box-shadow: 0px 1.48787px 16.3666px rgba(101, 84, 175, 0.1), 0px 2.97574px 4.46362px rgba(238, 238, 238, 0.16);
  border-radius: 5px;
}
</style>
