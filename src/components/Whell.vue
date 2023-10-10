<template>
  <div class="fw-container">
    <div
      class="fw-wheel"
      :style="rotateStyle"
      @transitionend="onRotateEnd"
      @webkitTransitionend="onRotateEnd"
    >
      <canvas
        v-if="type === 'canvas'"
        ref="wheel"
        :width="canvasConfig.radius * 2"
        :height="canvasConfig.radius * 2"
      />
      <slot name="wheel" v-else />
    </div>

    <div class="fw-btn">
      <div
        v-if="type === 'canvas'"
        class="fw-btn__btn"
        :style="{
          width: canvasConfig.btnWidth + 'px',
          height: canvasConfig.btnWidth + 'px',
        }"
        @click="handleClick"
      >
        {{ canvasConfig.btnText }}
      </div>
      <div v-else class="fw-btn__image" @click="handleClick">
        <slot name="button" />
      </div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import _sumBy from "lodash/sumBy";
import _random from "lodash/random";

function getStrArray(str, len) {
  const arr = [];
  while (str !== "") {
    let text = str.substr(0, len);
    if (str.charAt(len) !== "" && str.charAt(len) !== " ") {
      // Если следующая строка существует и первый символ следующей
      // строки не является пробелом
      const index = text.lastIndexOf(" ");
      if (index !== -1) text = text.substr(0, index);
    }
    str = str.replace(text, "").trim();
    arr.push(text);
  }
  return arr;
}

const canvasDefaultConfig = {
  radius: 250, //радиус
  textRadius: 190, // Расстояние между призовой позицией и центром круга
  textLength: 6, // Текст приза 1 строка из нескольких символов, до 2 строк
  textDirection: "horizontal", // Направление текста приза
  lineHeight: 20, // высота строки текста
  borderWidth: 1, // круглая внешняя граница
  borderColor: "red", // цвет внешней границы
  btnText: "kapibara", // текст кнопки запуска
  btnWidth: 200, // ширина кнопки
  fontSize: 20, // Размер приза
};

export default {
  name: "Whell",

  props: {
    type: {
      type: String,
      default: "canvas",
    },
    canvas: {
      type: Object,
      default: () => canvasDefaultConfig,
    },
    disabled: {
      type: Boolean,
      default: false, // следует ли отключить
    },
    useWeight: {
      type: Boolean,
      default: false, // вероятность по весу
    },
    prizes: {
      type: Array,
      required: true,
      default: () => [], // Список призов
    },
    verify: {
      type: Boolean,
      default: false, // Включить ли проверку
    },
    prizeId: {
      type: Number,
      default: 0, // Когда не используется 0, когда используются другие значения, результатом вращения является приз этого Id, который можно изменить во время вращения
    },
    angleBase: {
      type: Number,
      default: 10, // Основание угла поворота, количество витков 360*10
    },
    duration: {
      type: Number,
      default: 6000, // Время от одного оборота, в миллисекундах
    },
    timingFun: {
      type: String,
      default: "cubic-bezier(0.36, 0.95, 0.64, 1)", // Функция времени перехода вращения поворотного стола
    },
  },

  data() {
    return {
      isRotating: false, // он крутится?
      rotateEndDeg: 0, // угол на который поворачивается барабан
      prizeRes: {}, // Результаты вращения поворотного стола
    };
  },

  created() {
    this.checkProbability();
  },

  mounted() {
    if (this.type === "canvas") this.drawCanvas();
  },

  computed: {
    canRotate() {
      return (
        !this.disabled && !this.isRotating && this.probabilityTotal === 100
      );
    },
    probabilityTotal() {
      if (this.useWeight) return 100;
      return _sumBy(this.prizes, (row) => row.probability || 0);
    },
    canvasConfig() {
      return Object.assign(canvasDefaultConfig, this.canvas);
    },
    prizesIdArr() {
      const idArr = [];
      this.prizes.forEach((row) => {
        const count = this.useWeight
          ? row.weight || 0
          : (row.probability || 0) * this.decimalSpaces;
        const arr = new Array(count).fill(row.id);
        idArr.push(...arr);
      });
      return idArr;
    },
    rotateBase() {
      let angle = this.angleBase * 360;
      if (this.angleBase < 0) angle -= 360;
      return angle;
    },
    decimalSpaces() {
      if (this.useWeight) return 0;
      const sortArr = [...this.prizes].sort((a, b) => {
        const aRes = String(a.probability).split(".")[1];
        const bRes = String(b.probability).split(".")[1];
        const aLen = aRes ? aRes.length : 0;
        const bLen = bRes ? bRes.length : 0;
        return bLen - aLen;
      });
      const maxRes = String(sortArr[0].probability).split(".")[1];
      const idx = maxRes ? maxRes.length : 0;
      return [1, 10, 100, 1000, 10000][idx > 4 ? 4 : idx];
    },
    rotateStyle() {
      return {
        "-webkit-transform": `rotateZ(${this.rotateEndDeg}deg)`,
        transform: `rotateZ(${this.rotateEndDeg}deg)`,
        "-webkit-transition-duration": `${this.rotateDuration}s`,
        "transition-duration": `${this.rotateDuration}s`,
        "-webkit-transition-timing-function:": this.timingFun,
        "transition-timing-function": this.timingFun,
      };
    },
    rotateDuration() {
      return this.isRotating ? this.duration / 1000 : 0;
    },
  },

  methods: {
    handleClick() {
      if (!this.canRotate) return;
      if (this.verify) {
        this.$emit("rotate-start", this.onRotateStart);
        return;
      }
      this.$emit("rotate-start");
      this.onRotateStart();
    },
    onRotateStart() {
      this.isRotating = true;
      const prizeId = this.prizeId || this.getRandomPrize();
      this.rotateEndDeg = this.rotateBase + this.getTargetDeg(prizeId);
    },
    getRandomPrize() {
      const len = this.prizesIdArr.length;
      const prizeId = this.prizesIdArr[_random(0, len - 1)];
      return prizeId;
    },
    getTargetDeg(prizeId) {
      const angle = 360 / this.prizes.length;
      const num = this.prizes.findIndex((row) => row.id === prizeId);
      this.prizeRes = this.prizes[num];
      return 360 - (angle * num + angle / 2);
    },
    checkProbability() {
      if (this.probabilityTotal !== 100) {
        throw new Error("Prizes Is Error: Sum of probabilities is not 100!");
      }
      return true;
    },
    drawCanvas() {
      const canvasEl = this.$refs.wheel;
      if (canvasEl.getContext) {
        const {
          radius,
          textRadius,
          borderWidth,
          borderColor,
          fontSize,
        } = this.canvasConfig;
        // Рассчитать угол окружности исходя из количества призов
        const arc = Math.PI / (this.prizes.length / 2);
        const ctx = canvasEl.getContext("2d");
        // Очистить прямоугольник внутри заданного прямоугольника
        ctx.clearRect(0, 0, radius * 2, radius * 2);
        // Свойство задает или возвращает цвет,
        // градиент или узор, используемые для обводки.
        ctx.strokeStyle = borderColor;
        ctx.lineWidth = borderWidth * 2;
        // Свойство задает или возвращает текущее свойство
        // шрифта текстового содержимого на холсте.
        ctx.font = `${fontSize}px Arial`;
        this.prizes.forEach((row, i) => {
          const angle = i * arc - Math.PI / 2;
          ctx.fillStyle = row.bgColor;
          ctx.beginPath();
          // arc(x, y, r, Начальный угол, Конечный угол, Направление рисования)
          // для создания дуг/кривых (для создания кругов или частичных кругов)
          ctx.arc(
            radius,
            radius,
            radius - borderWidth,
            angle,
            angle + arc,
            false
          );
          ctx.stroke();
          ctx.arc(radius, radius, 0, angle + arc, angle, true);
          ctx.fill();
          // Заблокировать холст (чтобы сохранить предыдущее состояние холста)
          ctx.save();
          // Начинается розыгрыш приза
          ctx.fillStyle = row.color;
          // метод переназначения позиции (0, 0) на холсте
          ctx.translate(
            radius + Math.cos(angle + arc / 2) * textRadius,
            radius + Math.sin(angle + arc / 2) * textRadius
          );
          // способ повернуть текущий рисунок
          this.drawPrizeText(ctx, angle, arc, row.name);
          // Возвращает (настраивает) текущий холст в предыдущее состояние save()
          ctx.restore();
          // Конец розыгрыша призов
        });
      }
    },
    onRotateEnd() {
      this.isRotating = false;
      this.rotateEndDeg %= 360;
      this.$emit("rotate-end", this.prizeRes);
    },
    drawPrizeText(ctx, angle, arc, name) {
      const { lineHeight, textLength, textDirection } = this.canvasConfig;
      // Приведенный ниже код воспроизводит различные эффекты, такие
      // как шрифты, цвета и эффекты изображения, в зависимости от
      // типа приза и длины названия приза. (Конкретные изменения
      // в зависимости от реальной ситуации)
      const content = getStrArray(name, textLength);
      if (content === null) return;
      textDirection === "vertical"
        ? ctx.rotate(angle + arc / 2 + Math.PI)
        : ctx.rotate(angle + arc / 2 + Math.PI / 2);
      content.forEach((text, idx) => {
        let textX = -ctx.measureText(text).width / 2;
        let textY = (idx + 1) * lineHeight;
        if (textDirection === "vertical") {
          textX = 0;
          textY = (idx + 1) * lineHeight - (content.length * lineHeight) / 2;
        }
        ctx.fillText(text, textX, textY);
      });
    },
  },
};
</script>

<style scoped>
.fw-container {
  position: relative;
  display: inline-block;
  font-size: 0;
  overflow: hidden;
}

.fw-container canvas,
img {
  display: block;
  width: 100%;
}

.fw-btn {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.fw-btn__btn {
  position: relative;
  width: 100%;
  height: 100%;
  background: #fff;
  border: 6px solid #fff;
  border-radius: 50%;
  background: rgb(0, 0, 0);
  color: #fff;
  text-shadow: 0 0 7px #fff, 0 0 10px #fff, 0 0 21px #fff,
    0 0 42px rgb(158, 9, 228), 0 0 82px rgb(167, 4, 243),
    0 0 92px rgb(183, 2, 255), 0 0 102px rgb(212, 0, 255),
    0 0 151px rgb(239, 5, 247);
  text-align: center;
  font-size: 30px;
  font-weight: bold;
  line-height: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}
.fw-btn__btn:after {
  content: "";
  display: block;
  width: 0;
  height: 0;
  border-left: 18px solid transparent;
  border-right: 18px solid transparent;
  border-bottom: 22px #fff solid;
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
}

.fw-btn__btn:before {
  content: "";
  display: block;
  width: 0;
  height: 0;
  border-left: 12px solid transparent;
  border-right: 12px solid transparent;
  border-bottom: 18px rgb(247, 232, 23) solid;
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translate(-50%) translateY(6px);
  z-index: 10;
}
</style>
