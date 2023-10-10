<template>
  <div id="app">
    <Whell
      style="width: 500px; max-width: 100%"
      :prizes="prizesCanvas"
      :verify="canvasVerify"
      :canvas="canvasOptions"
      @rotateStart="onCanvasRotateStart"
      @rotateEnd="onRotateEnd"
    />

    <ul>
      <li v-for="item in prizeList" v-bind:key="item.id">
        {{ item.name }}
      </li>
    </ul>

    <Whell
      style="width: 500px; max-width: 100%"
      type="image"
      :useWeight="true"
      :verify="canvasVerify"
      :angleBase="-3"
      :prizes="prizesImage"
      @rotateStart="onImageRotateStart"
      @rotateEnd="onRotateEnd"
    >
      <img
        slot="wheel"
        src="./assets/Wheel.svg"
        style="transform: rotateZ(50deg)"
      />
      <img slot="button" src="./assets/button.svg" style="width: 180px" />
    </Whell>
  </div>
</template>

<script>
import Whell from "./components/Whell.vue";
export default {
  name: "App",
  components: {
    Whell,
  },
  data() {
    return {
      prizesImage: [
        {
          id: 1,
          value: "Blue's value",
          weight: 1,
        },
        {
          id: 2,
          value: "Red's value",
          weight: 1,
        },
        {
          id: 3,
          value: "Yellow's value",
          weight: 1,
        },
        {
          id: 4,
          value: "Yellow's value",
          weight: 1,
        },
        {
          id: 5,
          value: "Yellow's value",
          weight: 1,
        },
      ],

      prizesCanvas: [
        {
          id: 1,
          name: "Blue",
          value: "Blue's value",
          bgColor: "#542A68",
          color: "#ffffff",
          probability: 20,
        },
        {
          id: 2,
          name: "Red",
          value: "Red's value",
          bgColor: "#BE71FF",
          color: "#ffffff",
          probability: 20,
        },
        {
          id: 3,
          name: "Yellow",
          value: "Yellow's value",
          bgColor: "#e524d2",
          color: "#ffffff",
          probability: 20,
        },
        {
          id: 4,
          name: "Yellow",
          value: "Yellow's value",
          bgColor: "#e477b7",
          color: "#ffffff",
          probability: 20,
        },

        {
          id: 5,
          name: "Kapi",
          value: "Yellow's value",
          bgColor: "#FED601",
          color: "#ffffff",
          probability: 20,
        },
        {
          id: 6,
          name: "Blue",
          value: "Blue's value",
          bgColor: "#542A68",
          color: "#ffffff",
          probability: 20,
        },
        {
          id: 7,
          name: "Blue",
          value: "Blue's value",
          bgColor: "#542A68",
          color: "#ffffff",
          probability: 20,
        },
      ],

      canvasVerify: false, // Нужно ли делать проверку

      canvasOptions: {
        btnWidth: 160,
        borderColor: "#111",
        borderWidth: 6,
        lineHeight: 30,
        boxShadow: "120px 80px 40px 20px #0ff",
      },

      prizeId: 0,

      prizeList: [],
    };
  },
  methods: {
    onCanvasRotateStart(rotate) {
      console.log("onCanvasRotateStart");
    },
    onRotateEnd(prize) {
      this.prizeList.push(prize.name);
      alert(prize);
    },

    onImageRotateStart() {
      console.log("onImageRotateStart");
    },
  },
};
</script>

<style>
</style>
