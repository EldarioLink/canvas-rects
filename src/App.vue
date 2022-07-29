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
      updatedRectList: [],
      newArr: [],
      toSave: [],
      gapArr: [],
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

      // const newArr = this.updatedRectList.map((element) => {
      //   if (element.id === changedRect.attrs.id) {
      //     return {
      //       ...element,
      //       x: changedRect.absolutePosition().x,
      //       y: changedRect.absolutePosition().y,
      //       id: changedRect.attrs.id,
      //     };
      //   } else {
      //     let changedRectCoords = {
      //       x: changedRect.absolutePosition().x,
      //       y: changedRect.absolutePosition().y,
      //       id: changedRect.attrs.id,
      //     };
      //     return changedRectCoords;
      //   }
      // });
      // let isExist =
      //   this.updatedRectList.length > 0
      //     ? this.updatedRectList.find((item) => {
      //         console.log("ITEM", item);

      //         console.log(item.id, changedRect.attrs.id);
      //         return item.id === changedRect.attrs.id;
      //       })
      //     : null;

      let isExist = this.updatedRectList.filter((item) => {
        console.log("ITEM", item);

        console.log(item.id, changedRect.attrs.id);
        return item.id === changedRect.attrs.id;
      }).length;

      console.log("esixt", isExist);

      // eslint-disable-next-line no-extra-boolean-cast
      if (isExist) {
        this.updatedRectList = this.updatedRectList.map((element) => {
          console.log("check", element.id, changedRect.attrs.id);

          if (element.id === changedRect.attrs.id) {
            console.log("Bingo");
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
        console.log("else");

        let changedRectCoords = {
          x: changedRect.absolutePosition().x,
          y: changedRect.absolutePosition().y,
          id: changedRect.attrs.id,
        };
        this.updatedRectList.push(changedRectCoords);
      }
      console.log("updatedRectList", this.updatedRectList);

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
      console.log("save");

      // this.rectsList = this.rectsList.map((item) => {
      //   this.updatedRectList.forEach((el) => {
      //     console.log(item.id, el.id);
      //     if (item.id === el.id) {
      //       (item.x = el.x), (item.y = el.y);
      //     }
      //   });
      // });

      this.toSave = JSON.parse(JSON.stringify(this.rectsList));

      // this.toSave = this.toSave.map((el) => {
      //   this.updatedRectList.forEach((item) => {
      //     if (el.id === item.id) {
      //       console.log("HPPPPP");
      //       return {
      //         ...item,
      //         x: item.x,
      //         y: item.y,
      //         id: item.id,
      //       };
      //     } else {
      //       return item;
      //     }
      //   });
      // });
      // this.toSave.forEach((item) => {
      //   this.updatedRectList.forEach((el) => {
      //     console.log("fpre", item.id, el.id);
      //     if (item.id === el.id) {
      //       this.gapArr.push({
      //         ...item,
      //         x: el.x,
      //         y: el.y,
      //       });
      //     } else {
      //       this.gapArr.push(item);
      //     }
      //   });
      // });

      /// нужно обединить toSave и updatedRectList
      // this.gapArr = this.toSave.map((el) => {
      //   return this.updatedRectList.map((item) => {
      //     if (el.id === item.id) {
      //       console.log("equal");
      //       return {
      //         ...el,
      //         x: item.x,
      //         y: item.y,
      //         id: item.id,
      //       };
      //     } else {
      //       console.log("else");

      //       return el;
      //     }
      //   });
      // });
      this.gapArr = this.toSave.map((el) => {
        let filteredId = this.updatedRectList.filter((item) => {
          return el.id === item.id;
        });
        console.log("filteredId", filteredId, filteredId.x, filteredId.y);
        if (filteredId.length > 0) {
          console.log("equal");
          return {
            ...el,
            x: filteredId[0].x,
            y: filteredId[0].y,
            id: filteredId[0].id,
          };
        } else {
          console.log("else");

          return el;
        }
      });

      console.log("to save", this.toSave);
      console.log("updated", this.updatedRectList);

      console.log("to result", this.gapArr);

      console.log("rect", this.rectsList);
      localStorage.setItem("storage", JSON.stringify(this.gapArr));
      localStorage.setItem("connections", JSON.stringify(this.connectors));
    },
    restore() {
      console.log("rest");
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
