<template>
  <div id="app">
    <div class="menu_btns">
      <button class="button-13" role="button" @click="addRect">Добавить</button>

      <button class="button-13" role="button" @click="deleteRects">
        Очистить
      </button>
      <button class="button-13" role="button" @click="save">Сохранить</button>
      <button class="button-13" role="button" @click="restore">
        Восстановить
      </button>
    </div>

    <v-stage ref="stage" :config="stageSize">
      <v-layer ref="layer">
     
        <v-group
          v-for="item in rectsList"
          :key="item.id"
          ref="group"
          @dragmove="updateCoords"
          :config="{ draggable: isChosed ? false : true }"
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
           <v-line
          v-for="line in connectors"
          :key="line.id"
          :config="{
            stroke: 'black',
            strokeWidth: 4,
            points: line.coords,
          }"
        />
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
const height = window.innerHeight - 40;

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
      fromTarget: null,
      toTarget: null,
      parentFromId: null,
      parentToId: null,
      isChosed: null,
      updatedRectList: [],
      toSave: [],
      gapArr: [],
    };
  },
  methods: {
    addRect() {
      let xPos = Math.random() * width;
      let yPos = Math.random() * height;

      this.rectsList.push({
        id: Date.now() + Math.random(),
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
          this.isChosed = null
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
          parentFromId: this.parentFromId,
          parentToId: this.parentToId,
        };
        this.connectors.push(newConnector);

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
      let changedId = e.target.children.filter((circle) => {
        // eslint-disable-next-line no-undef
        return circle instanceof Konva.Circle;
      });

      let changedRect = e.target.children.filter((rect) => {
        // eslint-disable-next-line no-undef
        return rect instanceof Konva.Rect;
      })[0];

      let isExist = this.updatedRectList.filter((item) => {
        return item.id === changedRect.attrs.id;
      }).length;

      // eslint-disable-next-line no-extra-boolean-cast
      if (isExist) {
        this.updatedRectList = this.updatedRectList.map((element) => {
          if (element.id === changedRect.attrs.id) {
            return {
              x: changedRect.absolutePosition().x,
              y: changedRect.absolutePosition().y,
              id: changedRect.attrs.id,
            };
          } else {
            return element;
          }
        });
      } else {
        let changedRectCoords = {
          x: changedRect.absolutePosition().x,
          y: changedRect.absolutePosition().y,
          id: changedRect.attrs.id,
        };
        this.updatedRectList.push(changedRectCoords);
      }

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
    save() {
      if(!this.rectsList.length){
        return
      }
      this.toSave = JSON.parse(JSON.stringify(this.rectsList));

      this.gapArr = this.toSave.map((el) => {
        let filteredId = this.updatedRectList.filter((item) => {
          return el.id === item.id;
        });
        if (filteredId.length > 0) {
          return {
            ...el,
            x: filteredId[0].x,
            y: filteredId[0].y,
            id: filteredId[0].id,
          };
        } else {
          return el;
        }
      });
      localStorage.setItem("storage", JSON.stringify(this.gapArr));
      localStorage.setItem("connections", JSON.stringify(this.connectors));
    },
    restore() {
      let data = localStorage.getItem("storage") || "[]";
      let conData = localStorage.getItem("connections") || "[]";

      this.rectsList = JSON.parse(data);
      this.connectors = JSON.parse(conData);
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
    let deletedId;

    let currentShapeId;
    document.getElementById("delete-button").addEventListener("click", () => {
      this.connectors = this.connectors.filter((item) => {
        return item.parentFromId != deletedId && item.parentToId != deletedId;
      });
      currentShape.removeChildren();
      this.isChosed = null
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
      deletedId = e.target.attrs.id;
      currentShape = e.target.parent;
      currentShapeId = currentShape.children[0].attrs.id;

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
 @import '@/css/main.css';
</style>
