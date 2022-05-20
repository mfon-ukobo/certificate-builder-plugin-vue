<template>
  <div class="cb_container">
    <LeftSidebar :placeholdersConfig="placeholders" />

    <div class="cb_workspace_area">
      <div class="cb_workspace" @drop="handleWidgetDrop($event)" @dragover.prevent @dragenter.prevent></div>
    </div>

    <RightSidebar :selectedItem="selectedItem" :propertySidebarConfig="propertySidebarConfig" />
  </div>
</template>

<script>
import LeftSidebar from './components/LeftSidebar.vue'
import RightSidebar from './components/RightSidebar.vue'

import PROPERTY_ITEMS_TYPES from './constants/PropertySidebarItemTypes.js'

export default {
  components: {
    LeftSidebar,
    RightSidebar
  },
  props: {
    placeholdersConfig: Array,
    options: {
      type: Object,
      default() {
        return {
          container: 'cb_container',
          workspaceClass: 'cb_workspace',
          workspaceContainerClass: 'cb_workspace_container',
          workspaceItemClass: 'cb_workspace_item',
          activeWorkspaceItemClass: 'cb_is_current_workspace_item',
          widgetsContainerClass: 'cb_widgets_container',
          widgetContainerClass: 'cb_widget_container',
          widgetClass: 'cb_widget',
          placeholdersContainerClass: 'cb_placeholders_container',
          placeholders: [],
          imageUploadHandler: function (e) {
            console.log("Call API");
            let file = e.target.files[0];
            var bgUrl = URL.createObjectURL(file);
            this.workspace.style.backgroundImage = `url(${bgUrl})`;
          }
        }
      }
    }
  },
  data() {
    return {
      placeholders: [
        {
          id: 'w-1',
          name: 'First Name',
          placeholder: "first_name"
        },
        {
          id: 'w-2',
          name: 'Last Name',
          placeholder: "last_name"
        },
        {
          id: 'w-3',
          name: 'Score',
          placeholder: "score"
        }
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
                value: "12px"
              },
              {
                name: "14",
                value: "14px"
              },
              {
                name: "16",
                value: "16px"
              },
              {
                name: "18",
                value: "18px"
              }
            ]
          },
          {
            name: "Font Family",
            property: "font-family",
            type: PROPERTY_ITEMS_TYPES.dropdown,
            options: [
              {
                name: "Arial",
                value: "arial"
              },
              {
                name: "Consolas",
                value: "consolas"
              },
              {
                name: "Sans Serif",
                value: "sans-serif"
              },
              {
                name: "Times New Roman",
                value: "times new roman"
              }
            ]
          },
          {
            name: "Font Styling",
            type: PROPERTY_ITEMS_TYPES.checkbox,
            properties: [
              {
                name: "B",
                property: "font-weight",
                value: "bold"
              },
              {
                name: "U",
                property: "text-decoration",
                value: "underline"
              },
              {
                name: "I",
                property: "font-style",
                value: "italic"
              }
            ]
          },
          {
            name: "Color",
            property: "color",
            type: PROPERTY_ITEMS_TYPES.color
          }
        ]
      },
      dragged: null,
      workspace: null,
      selectedItem: null
    }
  },
  methods: {
    handleWidgetDrop(e) {
      let dropElement;

      if (this.dragged == null) {
        let id = e.dataTransfer.getData('widget');

        let placeholder = this.placeholders.find(x => x.id == id);

        dropElement = document.createElement("span");
        dropElement.draggable = true;

        dropElement.classList.add("cb_workspace_item");

        dropElement.style.position = "absolute";
        dropElement.textContent = `{{${placeholder.placeholder}}}`;

        dropElement.addEventListener('dragstart', this.handleWorkspaceItemDragStart);
        dropElement.addEventListener('click', this.handleWorkspaceItemClick);
        document.addEventListener('keydown', this.handleArrowKeyDown);
      }
      else {
        dropElement = this.dragged;
        this.dragged = null;
      }

      e.target.append(dropElement);

      let elWidth = dropElement.offsetWidth;
      let elHeight = dropElement.offsetHeight;

      dropElement.style.top = `${e.layerY - (elHeight / 2)}px`;
      dropElement.style.left = `${e.layerX - (elWidth / 2)}px`;
    },
    handleWorkspaceItemDragStart(e) {
      this.dragged = e.target;
    },
    handleWorkspaceItemClick(e) {
      this.selectItem(e.target);

      var workspaceItems = document.getElementsByClassName(this.options.workspaceItemClass);

      for (let item of workspaceItems) {
        item.classList.remove(this.options.activeWorkspaceItemClass);
      }

      e.target.classList.add(this.options.activeWorkspaceItemClass);
    },
    selectItem(element) {
      this.selectedItem = element;
    },
    buildPropertySidebar(element) {
      this.rightSidebar.innerHTML = null;
      for (let item of this.options.propertySidebar) {
        var container = document.createElement("div");
        container.background = "white";
        container.classList.add("card", "card-body", "mb-3", "m-3");

        var label = document.createElement("label");
        label.textContent = item.name;
        container.append(label);

        let input;

        switch (item.type) {
          case PROPERTY_ITEMS_TYPES.color:
            input = document.createElement("input");
            input.type = "color";
            input.classList.add("form-control");

            input.addEventListener('change', (e) => {
              this.currentElement.style[item.property] = e.target.value;
            });
            break;
          case PROPERTY_ITEMS_TYPES.dropdown:
            input = document.createElement("select");
            input.classList.add("form-control");
            if (item.options) {
              for (let option of item.options) {
                let optionElement = document.createElement("option");
                optionElement.textContent = option.name;
                optionElement.value = option.value;
                if (element.style[item.property] == option.value) {
                  optionElement.selected = true;
                }

                input.appendChild(optionElement);
              }
            }
            input.addEventListener('change', (e) => {
              this.currentElement.style[item.property] = e.target.value;
            });
            break;
          case PROPERTY_ITEMS_TYPES.checkbox:
            input = document.createElement("div");
            input.classList.add("btn-group", "btn-group-toggle");

            for (let property of item.properties) {
              let groupItem = document.createElement("label");
              groupItem.classList.add("btn", "btn-primary");
              groupItem.dataset.value = property.value;
              groupItem.dataset.property = property.property;

              let groupItemInput = document.createElement("input");
              groupItemInput.value = property.value;
              groupItemInput.type = "checkbox";
              groupItemInput.name = item.name;

              groupItem.title = property.name;

              var textNode = document.createTextNode(property.name);
              groupItem.append(textNode);

              groupItem.addEventListener("click", (e) => {
                let propertyName = e.target.dataset.property;
                let propertyValue = e.target.dataset.value;

                if (this._currentElement.style[propertyName] != propertyValue) {
                  this.currentElement.style[propertyName] = propertyValue;
                }
                else {
                  this.currentElement.style[propertyName] = "";
                }
              });

              input.appendChild(groupItem);
            }
            break;
          default:
            break;
        }

        container.append(input);

        this.rightSidebar.append(container);
      }
    }
  }
}
</script>

<style scoped>
.cb_container {
  display: flex;
  align-items: stretch;
}

.cb_workspace_area {
  width: 100%;
  flex-basis: 1;
  min-height: 300px;
  background-color: lightgray;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cb_workspace {
  min-width: 50%;
  min-height: 50%;
  background-color: white;
  box-shadow: 0px 0px 5px -3px;
}

.cb_workspace_item {
  height: auto;
  line-height: 1.5;
}

.cb_workspace_item.cb_is_current_workspace_item {
  background-color: lightgray!important;
  box-shadow: 0px 0px 5px -1px;
}
</style>