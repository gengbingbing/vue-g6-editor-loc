<template>
  <div class="toolbar">
    <a href="https://github.com/gengbingbing/vue-g6-editor-loc">
      <svg style="width: 16px; height: 16px;" t="1589093340467" class="iconSvg" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3031" width="200" height="200"><path d="M510.976182 93.63661l0-1.272993 0 0L510.976182 93.63661 510.976182 93.63661zM391.080511 961.609021 226.178811 961.609021c-16.537651 0-29.92966-13.406335-29.92966-29.943986L196.249151 541.973662 76.352457 541.973662c-8.268314 0-14.987343-6.710842-14.987343-14.987343 0-4.874007 2.326998-9.198499 5.928013-11.935843l-0.029676 0L497.906515 70.051449c2.561335-4.573155 7.463995-7.661494 13.068643-7.661494 5.635347 0 10.552334 3.118014 13.113669 7.728008l430.569386 444.919209c3.601015 2.737344 5.92699 7.077186 5.92699 11.95017 0 8.276501-6.716982 14.987343-14.987343 14.987343l0 0.028653L795.728528 542.003338l0 389.662721c0 16.537651-13.362332 29.943986-29.92966 29.943986L600.898191 961.610044 600.898191 736.768183c0-16.581654-13.450337-29.936823-30.061666-29.936823L421.142178 706.83136c-16.61133 0-30.061666 13.406335-30.061666 29.936823L391.080511 961.609021z" p-id="3032"></path></svg>
    </a>
    <link
      rel="stylesheet"
      type="text/css"
      href="./css/icon.css"
    />
    <i
      class="command iconfont icon-undo"
      title="撤销"
      :class="undoList.length>0?'':'disable'"
      @click="handleUndo"
    ></i>
    <i
      class="command iconfont icon-redo"
      title="重做"
      :class="redoList.length>0?'':'disable'"
      @click="handleRedo"
    ></i>
    <span class="separator"></span>
    <!-- <i data-command="copy" class="command iconfont icon-copy-o disable" title="复制"></i>
    <i data-command="paste" class="command iconfont icon-paster-o disable" title="粘贴"></i>-->
    <i
      data-command="delete"
      class="command iconfont icon-delete-o"
      title="删除"
      :class="selectedItem?'':'disable'"
      @click="handleDelete"
    ></i>
    <span class="separator"></span>
    <i
      data-command="zoomIn"
      class="command iconfont icon-zoom-in-o"
      title="放大"
      @click="handleZoomIn"
    ></i>
    <i
      data-command="zoomOut"
      class="command iconfont icon-zoom-out-o"
      title="缩小"
      @click="handleZoomOut"
    ></i>
    <i
      data-command="autoZoom"
      class="command iconfont icon-fit"
      title="适应画布"
      @click="handleAutoZoom"
    ></i>
    <i
      data-command="resetZoom"
      class="command iconfont icon-actual-size-o"
      title="实际尺寸"
      @click="handleResetZoom"
    ></i>
    <span class="separator"></span>
    <i
      data-command="toBack"
      class="command iconfont icon-to-back"
      :class="selectedItem?'':'disable'"
      title="层级后置"
      @click="handleToBack"
    ></i>
    <i
      data-command="toFront"
      class="command iconfont icon-to-front"
      :class="selectedItem?'':'disable'"
      title="层级前置"
      @click="handleToFront"
    ></i>
    <span class="separator"></span>
    <span class="separator"></span>
    <i
      data-command="multiSelect"
      class="command iconfont icon-select"
      :class="multiSelect?'disable':''"
      title="多选"
      @click="handleMuiltSelect"
    ></i>
    <i
      data-command="addGroup"
      class="command iconfont icon-group"
      title="成组"
      :class="addGroup?'':'disable'"
      @click="handleAddGroup"
    ></i>
    <i data-command="unGroup" class="command iconfont icon-ungroup disable" title="解组"></i>
    <el-button @click="consoleData" type="primary">控制台（console）输出数据</el-button>
  </div>
</template>

<script>
import eventBus from "@/utils/eventBus";
import Util from "@antv/g6/src/util";
import { uniqueId, getBox } from "@/utils";
export default {
  data() {
    return {
      page: {},
      graph: {},
      redoList: [],
      undoList: [],
      editor: null,
      command: null,
      selectedItem: null,
      multiSelect: false,
      addGroup: false
    };
  },
  created() {
    this.init();
    this.bindEvent();
  },
  watch: {
    selectedItem(val) {
      if (val && val.length > 1) {
        this.addGroup = true;
      } else {
        this.addGroup = false;
      }
    }
  },
  methods: {
    init() {
      const { editor, command } = this.$parent;
      this.editor = editor;
      this.command = command;
    },
    bindEvent() {
      let self = this;
      eventBus.$on("afterAddPage", page => {
        self.page = page;
        self.graph = self.page.graph;
      });
      eventBus.$on("add", data => {
        this.redoList = data.redoList;
        this.undoList = data.undoList;
      });
       eventBus.$on("update", data => {
        this.redoList = data.redoList;
        this.undoList = data.undoList;
      });
       eventBus.$on("delete", data => {
        this.redoList = data.redoList;
        this.undoList = data.undoList;
      });
      eventBus.$on("updateItem", item => {
        this.command.executeCommand("update", [item]);
      });
      eventBus.$on("addItem", item => {
        this.command.executeCommand("add", [item]);
      });
      eventBus.$on("nodeselectchange", () => {
        this.selectedItem = this.graph.findAllByState("node", "selected");
        this.selectedItem = this.selectedItem.concat(
          ...this.graph.findAllByState("edge", "selected")
        );
      });
      eventBus.$on("deleteItem", () => {
        this.handleDelete();
      });
      eventBus.$on("muliteSelectEnd", () => {
        this.multiSelect = false;
        this.selectedItem = this.graph.findAllByState("node", "selected");
      });
    },
    handleUndo() {
      if (this.undoList.length > 0) this.command.undo();
    },
    handleRedo() {
      if (this.redoList.length > 0) this.command.redo();
    },
    handleDelete() {
      if (this.selectedItem.length > 0) {
        this.command.executeCommand("delete", this.selectedItem);
        this.selectedItem = null;
      }
    },
    getFormatPadding() {
      return Util.formatPadding(this.graph.get("fitViewPadding"));
    },
    getViewCenter() {
      const padding = this.getFormatPadding();
      const graph = this.graph;
      const width = this.graph.get("width");
      const height = graph.get("height");
      return {
        x: (width - padding[2] - padding[3]) / 2 + padding[3],
        y: (height - padding[0] - padding[2]) / 2 + padding[0]
      };
    },
    handleZoomIn() {
      const currentZoom = this.graph.getZoom();
      this.graph.zoomTo(currentZoom + 0.5, this.getViewCenter());
    },
    handleZoomOut() {
      const currentZoom = this.graph.getZoom();
      this.graph.zoomTo(currentZoom - 0.5, this.getViewCenter());
    },
    handleToBack() {
      if (this.selectedItem && this.selectedItem.length > 0) {
        this.selectedItem.forEach(item => {
          item.toBack();
          this.graph.paint();
        });
      }
    },
    handleToFront() {
      if (this.selectedItem && this.selectedItem.length > 0) {
        this.selectedItem.forEach(item => {
          if (item.getType() === "edge") {
            // const nodeGroup = this.graph.get("nodeGroup");
            // const edgeGroup = item.get("group");
            // nodeGroup.toFront();
            // edgeGroup.toFront()
          } else {
            item.toFront();
          }

          this.graph.paint();
        });
      }
    },
    handleAutoZoom() {
      this.graph.fitView(20);
    },
    handleResetZoom() {
      this.graph.zoomTo(1, this.getViewCenter());
    },
    handleMuiltSelect() {
      this.multiSelect = true;
      this.graph.setMode("mulitSelect");
    },
    handleAddGroup() {
      //TODO 这部分等阿里更新Group之后添加
      // const model = {
      //   id: "group" + uniqueId(),
      //   title: "新建分组"
      // };
      // // this.command.executeCommand("add", "group", model);
      // this.selectedItem.forEach(item => {
      //   console.log(item);
      // });
  
      //this.getPosition(this.selectedItem);
    },
    getPosition(items) {
      const boxList = [];
      items.forEach(item => {
        const box = item.getBBox();
        boxList.push(getBox(box.x, box.y, box.width, box.height));
      });
      let minX1, minY1, MaxX2, MaxY2;
      boxList.forEach(box => {
        if (typeof minX1 == "undefined") {
          minX1 = box.x1;
        }
        if (typeof minY1 == "undefined") {
          minY1 = box.y1;
        }
        if (typeof MaxX2 == "undefined") {
          MaxX2 = box.x2;
        }
        if (typeof MaxY2 == "undefined") {
          MaxY2 = box.y2;
        }
        if (minX1 > box.x1) {
          minX1 = box.x1;
        }
        if (minY1 > box.y1) {
          minY1 = box.y1;
        }
        if (MaxX2 < box.x2) {
          MaxX2 = box.x2;
        }
        if (MaxY2 < box.y2) {
          MaxY2 = box.y2;
        }
      });
      const width = MaxX2 - minX1,
        height = MaxY2 - minY1,
        x = minX1 + width / 2,
        y = minY1 + height / 2,
        id = "team" + uniqueId();
      const model = {
        id: id,
        width,
        height,
        x,
        y,
        shape: "teamNode"
      };
      this.command.executeCommand("add", model);
      // const item = this.graph.findById(id);
      // item.get("group").toBack();
      // const edgeGroup = this.graph.get("edgeGroup");
      // edgeGroup.toFront();
      // this.graph.paint();
    },

    consoleData() {
      // eslint-disable-next-line no-console
      console.log("编辑器数据===>", this.graph.save());
    }
  }
};
</script>


<style scoped>
.toolbar {
  box-sizing: border-box;
  padding: 8px 0px;
  width: 100%;
  border: 1px solid #e9e9e9;
  height: 42px;
  z-index: 3;
  box-shadow: 0px 8px 12px 0px rgba(0, 52, 107, 0.04);
  position: absolute;
}
.toolbar .command:nth-of-type(1) {
  margin-left: 24px;
}
.toolbar .command {
  box-sizing: border-box;
  width: 27px;
  height: 27px;
  margin: 0px 6px;
  border-radius: 2px;
  padding-left: 4px;
  display: inline-block;
  border: 1px solid rgba(2, 2, 2, 0);
}
.toolbar .command:hover {
  cursor: pointer;
  border: 1px solid #e9e9e9;
}
.toolbar .disable {
  color: rgba(0, 0, 0, 0.25);
}
.toolbar .separator {
  margin: 4px;
  border-left: 1px solid #e9e9e9;
}
.iconSvg {
    width: 16px;
    height: 16px;
    margin-left: 23px;
}
</style>