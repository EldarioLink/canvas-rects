<template>
  <div id="app">
    <button @click="addRect">Добавить</button>
    <hr />
    <button @click="deleteRects">Очистить</button>

    <v-stage ref="stage" :config="stageSize">
      <v-layer ref="layer">
        <v-group
          v-for="item in rectsList"
          :key="item.id"
          ref="group"
          :config="{ draggable: true }"
        >
          <v-rect
            ref="item.id"
            :config="{
              x: item.x,
              y: item.y,
              id: item.id,
              width: 100,
              height: 100,
              stroke: 'black',
              strokeWidth: 4,
            }"
          >
          </v-rect>
          <v-circle
            :config="{
              x: item.x + 51,
              y: item.y - 5,
              radius: 10,
              fill: 'black',
              draggable: false,
            }"
          />
          <v-circle
            :config="{
              x: item.x + 51,
              y: item.y + 105,
              radius: 10,
              fill: 'black',
              draggable: false,
            }"
          />
          <v-circle
            :config="{
              x: item.x + 104,
              y: item.y + 51,
              radius: 10,
              fill: 'black',
              draggable: false,
            }"
          />
          <v-circle
            :config="{
              x: item.x - 4,
              y: item.y + 51,
              radius: 10,
              fill: 'black',
              draggable: false,
            }"
          />
        </v-group>
      </v-layer>
    </v-stage>
    <div id="menu">
      <div>
        <button>1</button><button>2</button><button>3</button><button>4</button
        ><button id="delete-button">Delete</button>
      </div>
    </div>
  </div>
</template>

<script>
const width = window.innerWidth;
const height = window.innerHeight;

export default {
  data() {
    return {
      rectsList: [],
      stageSize: {
        width: width,
        height: height,
      },
      isDragging: false,
      nodeCounter: 0,
      nodeArray: [],
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
    openMenu() {
      console.log("click");
    },
    addRect() {
      this.rectsList.push({
        id: Math.round(Math.random() * 10000).toString(),
        x: Math.random() * width,
        y: Math.random() * height - 100,
      });
    },
    deleteRects() {
      this.rectsList = [];
    },
  },
  mounted() {
    const stage = this.$refs.stage.getNode();
    // setup menu
    var menuNode = document.getElementById("menu");
    let currentShape;

    document.getElementById("delete-button").addEventListener("click", () => {
      currentShape.removeChildren();
    });

    window.addEventListener("click", () => {
      // hide menu
      menuNode.style.display = "none";
    });

    stage.on("contextmenu", function (e) {
      // prevent default behavior
      e.evt.preventDefault();

      if (e.target === stage) {
        // if we are on empty place of the stage we will do nothing
        return;
      }
      console.log(e.target);
      currentShape = e.target.parent;
      // console.log("currentshape", group);
      // show menu
      menuNode.style.display = "initial";
      var containerRect = stage.container().getBoundingClientRect();
      menuNode.style.top =
        containerRect.top + stage.getPointerPosition().y + 4 + "px";
      menuNode.style.left =
        containerRect.left + stage.getPointerPosition().x + 4 + "px";
    });
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
#menu {
  display: none;
  position: absolute;
  width: 150px;
  background-color: white;
  box-shadow: 0 0 5px grey;
  border-radius: 3px;
}

#menu button {
  width: 100%;
  background-color: white;
  border: none;
  margin: 0;
  padding: 10px;
}

#menu button:hover {
  background-color: lightgray;
}
</style>
