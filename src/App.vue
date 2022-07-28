<template>
  <div id="app">
    <div>
      <button @click="addRect">Добавить</button>
      <button @click="deleteRects">Очистить</button>
      <button @click="saveLocal">Сохранить</button>
      <button @click="restoreLocal">Восстановить</button>
    </div>

    <v-stage ref="stage" :config="stageSize">
      <v-layer ref="layer">
        <v-line
          v-for="line in connectors"
          :key="line.id"
          :config="{
            stroke: 'black',
            points: line.points,
          }"
        />
        <v-group
          v-for="item in rectsList"
          :key="item.id"
          ref="group"
          :config="{ draggable: true }"
        >
          <v-rect
            ref="rect"
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
            v-for="el in getNodes.slice(0, item.activeNodes)"
            :key="el.id"
            @click="startLine"
            :config="{
              id: el.id,
              x: item.x + el.x,
              y: item.y + el.y,
              radius: 10,
              // eslint-disable-next-line vue/no-parsing-error
              fill: fromShapeId === el.id ? `red` : `green`,
              draggable: false,
            }"
          />
        </v-group>
      </v-layer>
    </v-stage>
    <div id="menu">
      <div>
        <button id="oneNode">1</button><button id="twoNode">2</button
        ><button id="threeNode">3</button><button id="fourNode">4</button
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
      connectors: [],
      drawningLine: false,
      fromShapeId: null,
    };
  },
  methods: {
    handleDragStart() {
      this.isDragging = true;
    },
    handleDragEnd() {
      this.isDragging = false;
    },
    addRect() {
      let xPos = Math.random() * width;
      let yPos = Math.random() * height;

      this.rectsList.push({
        id: Math.round(Math.random() * 10000).toString(),
        x: xPos >= width - 150 ? Math.random() * width : xPos,
        y: yPos >= height - 150 ? Math.random() * height : yPos,
        activeNodes: 1,
      });
    },
    deleteRects() {
      this.rectsList = [];
    },
    startLine(e) {
      // eslint-disable-next-line no-undef
      const onCircle = e.target instanceof Konva.Circle;
      if (!onCircle) {
        return;
      }
      console.log("fdfd");
      console.log("coords", e);

      if (this.fromShapeId) {
        console.log("coords", e.target.attrs.id);

        const newConnector = {
          from: e.target.x(),
          to: e.target.y(),
          id: this.connectors.length,
        };
        this.connectors.push(newConnector);
        this.fromShapeId = null;
      } else {
        this.fromShapeId = e.target.attrs.id;
      }
    },
    // handleMouseDown(e) {
    //   // eslint-disable-next-line no-undef
    //   const onCircle = e.target instanceof Konva.Circle;
    //   if (!onCircle) {
    //     return;
    //   }
    //   this.drawningLine = true;
    //   this.connections.push({
    //     id: Date.now(),
    //     points: [e.target.x(), e.target.y()],
    //   });
    // },
    // handleMouseUp(e) {
    //   // eslint-disable-next-line no-undef
    //   const onCircle = e.target instanceof Konva.Circle;
    //   if (!onCircle) {
    //     return;
    //   }
    //   this.drawningLine = false;
    //   const lastLine = this.connections[this.connections.length - 1];
    //   lastLine.points = [
    //     lastLine.points[0],
    //     lastLine.points[1],
    //     e.target.x(),
    //     e.target.y(),
    //   ];
    // },
    saveLocal() {
      // to do this
    },
    restoreLocal() {
      // to do this
    },
  },
  computed: {
    getNodes() {
      let rectNodes = [
        {
          // id: Math.round(Math.random() * 10000).toString(),
          x: +51,
          y: -5,
          radius: 10,
          fill: "black",
          draggable: false,
        },

        {
          // id: Math.round(Math.random() * 10000).toString(),
          x: -4,
          y: +51,
          radius: 10,
          fill: "black",
          draggable: false,
        },
        {
          // id: Math.round(Math.random() * 10000).toString(),
          x: +51,
          y: +105,
          radius: 10,
          fill: "black",
          draggable: false,
        },
        {
          // id: Math.round(Math.random() * 10000).toString(),
          x: +104,
          y: +51,
          radius: 10,
          fill: "black",
          draggable: false,
        },
      ];
      return rectNodes;
    },
  },
  mounted() {
    const stage = this.$refs.stage.getNode();

    // setup menu
    var menuNode = document.getElementById("menu");
    let currentShape;
    let currentShapeId;
    document.getElementById("delete-button").addEventListener("click", () => {
      currentShape.removeChildren();
    });

    document.getElementById("oneNode").addEventListener("click", () => {
      this.rectsList = this.rectsList.map((rect) =>
        rect.id === currentShapeId ? { ...rect, activeNodes: 1 } : rect
      );
    });

    document.getElementById("twoNode").addEventListener("click", () => {
      this.rectsList = this.rectsList.map((rect) =>
        rect.id === currentShapeId ? { ...rect, activeNodes: 2 } : rect
      );
    });

    document.getElementById("threeNode").addEventListener("click", () => {
      this.rectsList = this.rectsList.map((rect) =>
        rect.id === currentShapeId ? { ...rect, activeNodes: 3 } : rect
      );
    });

    document.getElementById("fourNode").addEventListener("click", () => {
      this.rectsList = this.rectsList.map((rect) =>
        rect.id === currentShapeId ? { ...rect, activeNodes: 4 } : rect
      );
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

      currentShape = e.target.parent;
      currentShapeId = currentShape.children[0].attrs.id;

      console.log("target", e.evt);

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
