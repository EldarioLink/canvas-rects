<template>
  <div id="app">
    <div class="menu_btns">
      <button class="button-13" role="button" @click="addRect">Добавить</button>

      <button class="button-13" role="button" @click="deleteRects">
        Очистить
      </button>
      <button class="button-13" role="button" @click="saveLocal">
        Сохранить
      </button>
      <button class="button-13" role="button" @click="restoreLocal">
        Восстановить
      </button>
    </div>

    <v-stage ref="stage" :config="stageSize">
      <v-layer ref="layer">
        <v-line
          v-for="line in connectors"
          :key="line.id"
          :config="{
            stroke: 'black',
            strokeWidth: 4,
            points: line.coords,
          }"
        />
        <v-group
          v-for="item in rectsList"
          :key="item.id"
          ref="group"
          @dragmove="updateCoords"
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
            ref="circle"
            v-for="el in item.nodes.slice(0, item.activeNodes)"
            :key="el.id"
            @click="startLine"
            :config="{
              id: el.id,
              x: item.x + el.x,
              y: item.y + el.y,
              radius: 10,
              // eslint-disable-next-line vue/no-parsing-error
              fill: isChosed === el.id ? `red` : `green`,
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
      fromTarget: null,
      toTarget: null,
      parentFromId: null,
      parentToId: null,
      isChosed: null,
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
        id: Math.round(Math.random() * 100000).toString(),
        x: xPos >= width - 150 ? Math.random() * width : xPos,
        y: yPos >= height - 150 ? Math.random() * height : yPos,
        activeNodes: 1,
        nodes: this.getNodes(),
      });
    },
    deleteRects() {
      this.rectsList = [];
      this.connectors = [];
    },
    startLine(e) {
      // eslint-disable-next-line no-undef
      const onCircle = e.target instanceof Konva.Circle;
      if (!onCircle) {
        return;
      }

      const stage = this.$refs.stage.getNode();
      if (this.isChosed) {
        this.toTarget = stage.getPointerPosition();
        this.parentToId = e.target.parent.children[0].attrs.id;

        if (this.parentFromId === this.parentToId) {
          return;
        }

        const newConnector = {
          coords: [
            this.fromTarget.x,
            this.fromTarget.y,
            this.toTarget.x,
            this.toTarget.y,
          ],
          id: this.connectors.length,
          from: this.isChosed,
          to: e.target.attrs.id,
        };
        this.connectors.push(newConnector);
        console.log("connectors", this.connectors);

        this.fromTarget = null;
        this.toTarget = null;
        this.isChosed = null;
      } else {
        this.fromTarget = stage.getPointerPosition();
        this.isChosed = e.target.attrs.id;
        this.parentFromId = e.target.parent.children[0].attrs.id;
      }
    },
    updateObjects() {},
    updateCoords(e) {
      let allCircles = e.target.children;
      console.log(e.target.children[1].absolutePosition());
      console.log(typeof allCircles);

      let changedId = e.target.children.filter((circle) => {
        // eslint-disable-next-line no-undef
        return circle instanceof Konva.Circle;
      });
      console.log(changedId[0].absolutePosition().x);

      this.connectors.map((item) => {
        changedId.forEach((element) => {
          if (item.from === element.attrs.id) {
            item.coords.splice(
              0,
              2,
              element.absolutePosition().x,
              element.absolutePosition().y
            );
          } else if (item.to === element.attrs.id) {
            item.coords.splice(
              2,
              2,
              element.absolutePosition().x,
              element.absolutePosition().y
            );
          }
        });
      });
    },
    saveLocal() {
      // to do this
    },
    restoreLocal() {
      // to do this
    },
    getNodes() {
      let rectNodes = [
        {
          id: Math.round(Math.random() * 10000).toString(),
          x: +51,
          y: -5,
          radius: 10,
          fill: "black",
          draggable: false,
        },

        {
          id: Math.round(Math.random() * 10000).toString(),
          x: -4,
          y: +51,
          radius: 10,
          fill: "black",
          draggable: false,
        },
        {
          id: Math.round(Math.random() * 10000).toString(),
          x: +51,
          y: +105,
          radius: 10,
          fill: "black",
          draggable: false,
        },
        {
          id: Math.round(Math.random() * 10000).toString(),
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
  computed: {},
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

.menu_btns {
  display: flex;
  justify-content: space-around;
}
.button-13 {
  background-color: #fff;
  border: 1px solid #d5d9d9;
  border-radius: 8px;
  box-shadow: rgba(213, 217, 217, 0.5) 0 2px 5px 0;
  box-sizing: border-box;
  color: #0f1111;
  cursor: pointer;
  display: inline-block;
  font-family: "Amazon Ember", sans-serif;
  font-size: 13px;
  line-height: 29px;
  padding: 0 10px 0 11px;
  position: relative;
  text-align: center;
  text-decoration: none;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  vertical-align: middle;
  width: 100px;
}

.button-13:hover {
  background-color: #f7fafa;
}

.button-13:focus {
  border-color: #008296;
  box-shadow: rgba(213, 217, 217, 0.5) 0 2px 5px 0;
  outline: 0;
}
</style>
