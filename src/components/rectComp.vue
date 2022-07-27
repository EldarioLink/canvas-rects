<template>
  <v-stage ref="stage" :config="stageSize">
    <v-layer ref="layer">
      <v-rect
        @dragstart="handleDragStart"
        @dragend="handleDragEnd"
        @click="openWindow"
        :config="{
          x: 50,
          y: 50,
          width: 100,
          height: 100,
          draggable: true,
          stroke: 'black',
          strokeWidth: 4,
        }"
      >
      </v-rect>
    </v-layer>
  </v-stage>
</template>

<script>
const width = window.innerWidth;
const height = window.innerHeight;

export default {
  data() {
    return {
      stageSize: {
        width: width,
        height: height,
      },
      isDragging: false,
      // isOpen: false,
      items: [
        { title: "Click Me" },
        { title: "Click Me" },
        { title: "Click Me" },
        { title: "Click Me 2" },
      ],
    };
  },
  methods: {
    handleDragStart() {
      this.isDragging = true;
    },
    handleDragEnd() {
      this.isDragging = false;
    },
    handleWindow() {
      console.log("click");
    },
    openWindow() {
      console.log("click");
      // this.isOpen != this.isOpen;
    },
  },
  mounted() {
    // const stage = this.$refs.stage.getNode();
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>
