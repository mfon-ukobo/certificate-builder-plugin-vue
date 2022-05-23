<template>
  <div class="cb_container">
    <LeftSidebar
      :placeholdersConfig="placeholders"
      @certificateImageUploaded="handleCertificateImageUpload"
    />

    <div class="cb_workspace_area_container">
      <div
        class="cb_workspace_area"
        @click="handleWorkspaceClick"
        @wheel="handleMouseWheel"
        ref="workspaceArea"
      >
        <div
          class="cb_workspace"
          @drop="handleWidgetDrop($event)"
          @dragover.prevent
          @dragenter.prevent
          ref="workspace"
        ></div>
      </div>
    </div>

    <RightSidebar
      :selectedItem="selectedItem"
      :propertySidebarConfig="propertySidebarConfig"
      :currentTab="rightSidebarCurrentTab"
      :paperSizes="paperSizes"
      @itemPropertyChanged="handlePropertyChanged"
      @paperSizeChanged="handlePaperSizeChanged"
      @backgroundImageChanged="handleBackgroundImageChanged"
    />
  </div>
</template>

<script>
import LeftSidebar from "./components/LeftSidebar.vue";
import RightSidebar from "./components/RightSidebar.vue";

import PROPERTY_ITEMS_TYPES from "./constants/PropertySidebarItemTypes.js";

export default {
  components: {
    LeftSidebar,
    RightSidebar,
  },
  props: {
    placeholdersConfig: Array,
    options: {
      type: Object,
      default() {
        return {
          container: "cb_container",
          workspaceClass: "cb_workspace",
          workspaceContainerClass: "cb_workspace_container",
          workspaceItemClass: "cb_workspace_item",
          activeWorkspaceItemClass: "active",
          widgetsContainerClass: "cb_widgets_container",
          widgetContainerClass: "cb_widget_container",
          widgetClass: "cb_widget",
          placeholdersContainerClass: "cb_placeholders_container",
          placeholders: [],
          imageUploadHandler: function (e) {
            console.log("Call API");
            let file = e.target.files[0];
            var bgUrl = URL.createObjectURL(file);
            this.workspace.style.backgroundImage = `url(${bgUrl})`;
          },
        };
      },
    },
    paperSizesConfig: {
      type: Array,
      default() {
        return [
          {
            name: "A4",
            width: 1056,
            height: 816,
          },
        ];
      },
    },
  },
  data() {
    return {
      placeholders: [
        {
          id: "w-1",
          name: "First Name",
          placeholder: "first_name",
        },
        {
          id: "w-2",
          name: "Last Name",
          placeholder: "last_name",
        },
        {
          id: "w-3",
          name: "Score",
          placeholder: "score",
        },
      ],
      propertySidebarConfig: {
        span: [
          {
            name: "Font Size",
            property: "font-size",
            type: PROPERTY_ITEMS_TYPES.dropdown,
            options: [
              {
                name: "12",
                value: "12px",
              },
              {
                name: "14",
                value: "14px",
              },
              {
                name: "16",
                value: "16px",
              },
              {
                name: "18",
                value: "18px",
              },
            ],
          },
          {
            name: "Font Family",
            property: "font-family",
            type: PROPERTY_ITEMS_TYPES.dropdown,
            options: [
              {
                name: "Arial",
                value: "arial",
              },
              {
                name: "Consolas",
                value: "consolas",
              },
              {
                name: "Sans Serif",
                value: "sans-serif",
              },
              {
                name: "Times New Roman",
                value: "times new roman",
              },
            ],
          },
          {
            name: "Font Styling",
            type: PROPERTY_ITEMS_TYPES.checkbox,
            properties: [
              {
                name: "B",
                property: "font-weight",
                value: "bold",
              },
              {
                name: "U",
                property: "text-decoration",
                value: "underline",
              },
              {
                name: "I",
                property: "font-style",
                value: "italic",
              },
            ],
          },
          {
            name: "Color",
            property: "color",
            type: PROPERTY_ITEMS_TYPES.color,
          },
        ],
      },
      paperSizes: Array,
      dragged: null,
      selectedItem: null,
      workspace: HTMLElement,
      rightSidebarCurrentTab: "workspace",
      workspaceScale: 1,
      scaleFactor: 0.1
    };
  },
  methods: {
    handleWidgetDrop(e) {
      let dropElement;

      if (this.dragged == null) {
        console.log("error");
        let id = e.dataTransfer.getData("widget");

        let placeholder = this.placeholders.find((x) => x.id == id);
        console.log({ placeholder, id });

        dropElement = document.createElement("span");
        dropElement.draggable = true;

        dropElement.classList.add("cb_workspace_item");

        dropElement.style.position = "absolute";
        dropElement.textContent = `{{${placeholder.placeholder}}}`;

        e.target.append(dropElement);

        dropElement.addEventListener(
          "dragstart",
          this.handleWorkspaceItemDragStart
        );
        dropElement.addEventListener("click", this.handleWorkspaceItemClick);
        document.addEventListener("keydown", this.handleArrowKeyDown);
      } else {
        dropElement = this.dragged;
        this.dragged = null;
      }

      this.selectItem(dropElement);

      if (this.selectedItem != e.target) {
        let elWidth = dropElement.offsetWidth;
        let elHeight = dropElement.offsetHeight;

        dropElement.style.top = `${e.layerY - elHeight / 2}px`;
        dropElement.style.left = `${e.layerX - elWidth / 2}px`;
      }
    },
    handleWorkspaceItemDragStart(e) {
      this.dragged = e.target;
    },
    handleWorkspaceItemClick(e) {
      e.stopPropagation();
      this.rightSidebarCurrentTab = "item";
      this.selectItem(e.target);
    },
    handleWorkspaceClick(e) {
      e.stopPropagation();
      this.rightSidebarCurrentTab = "workspace";
      this.selectItem(e.target);
    },
    selectItem(element) {
      this.selectedItem = element;

      if (element.classList.contains(this.options.workspaceClass)) {
        this.rightSidebarCurrentTab = "workspace";
        return;
      }

      this.rightSidebarCurrentTab = "item";
    },
    handlePropertyChanged(data) {
      let propertyData = data.propertyData;
      console.log({ SelectedItem: data.selectedItem });
      console.log({ ProprtyData: data.propertyData });
      console.log({ Value: data.value });

      this.selectedItem.style[propertyData.property] = data.value;
    },
    handleArrowKeyDown(e) {
      const VALID_KEY_CODES = [37, 38, 39, 40];
      const MOVEMENT_UNIT = 2;

      if (VALID_KEY_CODES.includes(e.keyCode)) {
        e.preventDefault();
        e.stopPropagation();
      }

      var currentElement = this.selectedItem;

      if (currentElement == null || currentElement == undefined) {
        return;
      }

      switch (e.keyCode) {
        case 37:
          currentElement.style.left =
            convertPxToNumber(currentElement.style.left) - MOVEMENT_UNIT + "px";
          break;
        case 38:
          currentElement.style.top =
            convertPxToNumber(currentElement.style.top) - MOVEMENT_UNIT + "px";
          break;
        case 39:
          currentElement.style.left =
            convertPxToNumber(currentElement.style.left) + MOVEMENT_UNIT + "px";
          break;
        case 40:
          currentElement.style.top =
            convertPxToNumber(currentElement.style.top) + MOVEMENT_UNIT + "px";
          break;

        default:
          break;
      }

      // Convert px to number
      function convertPxToNumber(px) {
        return Number.parseInt(px.replace("px", ""));
      }
    },
    handleCertificateImageUpload(data) {
      let workspace = this.$refs.workspace;
      workspace.style.backgroundImage = `url(${data})`;
    },
    handlePaperSizeChanged(paperConfig) {
      let workspace = this.$refs.workspace;
      workspace.style.height = paperConfig.height + "px";
      workspace.style.width = paperConfig.width + "px";
      this.workspaceScale = 1 - this.scaleFactor;
      console.log()
    },
    handleBackgroundImageChanged(data) {
      this.selectedItem.style.backgroundImage = `url(${data})`;
    },
    handleMouseWheel(e) {
      e.preventDefault();
      let scale = this.workspaceScale;
      scale += e.deltaY * -0.001;
      scale = Math.min(Math.max(0.125, scale), 4);

      let workspaceArea = this.$refs.workspaceArea;
      workspaceArea.style.transformOrigin = `50% 50%`;

      this.workspaceScale = scale;
    },
    addEventListeners() {
      let $this = this;

      document.addEventListener("click", function (e) {
        if (e.target.classList.contains($this.options.workspaceItemClass)) {
          $this.handleWorkspaceItemClick(e);
        } else if (e.target.classList.contains($this.options.workspaceClass)) {
          $this.handleWorkspaceClick(e);
        }
      });

      document.addEventListener("dragstart", function (e) {
        if (e.target.classList.contains($this.options.workspaceItemClass)) {
          $this.handleWorkspaceItemDragStart(e);
        }
      });

      document.addEventListener("drag", function(e) {
        if (e.target.classList.contains($this.options.workspaceItemClass)) {
          console.log(e);
        }
      })

      document.addEventListener("drop", function (e) {
        e.preventDefault();
        if (e.target.classList.contains($this.options.workspaceClass)) {
          //$this.handleWidgetDrop(e);
        }
      });
    },
  },
  watch: {
    selectedItem(item) {
      // remove active class from all workspace items
      var activeItems = document.getElementsByClassName(
        this.options.activeWorkspaceItemClass
      );

      for (let activeItem of activeItems) {
        activeItem.classList.remove(this.options.activeWorkspaceItemClass);
      }

      if (item.classList.contains("cb_workspace_item")) {
        item.classList.add(this.options.activeWorkspaceItemClass);
        return;
      }

      item.classList.add(this.options.activeWorkspaceItemClass);
    },
    workspaceScale(value) {
      let workspaceArea = this.$refs.workspaceArea;
      workspaceArea.style.transform = `scale(${value})`;
    },
  },
  created() {
    this.paperSizes = this.paperSizesConfig;

    this.addEventListeners();
  },
  mounted() {
    this.selectedItem = this.$refs.workspace;
  },
};
</script>

<style>
.cb_container {
  display: flex;
  align-items: stretch;
  max-height: 100%;
  overflow: auto;
}

.cb_workspace_area_container {
  height: 500px;
  width: 90%;
  overflow: auto;
  background-color: lightgray;
  border-radius: 24px;
}

.cb_workspace_area {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  padding: 30px;
  transform-origin: top left;
}

.cb_workspace {
  position: relative;
  width: 400px;
  height: 200px;
  background-color: white;
  background-size: 100% 100%;
  box-shadow: 0px 0px 5px -3px;
}

.cb_workspace.active {
  outline: 2px solid black;
}

.cb_workspace_item {
  height: auto;
  line-height: 1;
  padding: 4px !important;
  border-radius: 8px;
  cursor: grab;
}

.cb_workspace_item.active {
  padding: 2px;
  outline: 1px solid black;
}
</style>
