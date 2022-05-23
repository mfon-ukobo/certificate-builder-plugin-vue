<template>
  <div class="cb_right_sidebar_container">
    <div class="cb_right_sidebar">
      <div v-if="currentTab == 'item'">
        <div v-for="(propertyItem, index) in propertySidebarItems" :key="index">
          <div class="card mb-2">
            <div class="cb_property_item card-body">
              <label>{{ propertyItem.name }}</label>

              <div v-if="propertyItem.type == propertyTypes.dropdown">
                <select class="form-control" @change.prevent="handlePropertyChanged($event, propertyItem)">
                  <option :value="option.value" v-for="(option, index2) in propertyItem.options"
                    :key="propertyItem.property + index2">
                    {{ option.name }}
                  </option>
                </select>
              </div>

              <div v-if="propertyItem.type == propertyTypes.checkbox" class="btn-group btn-group-toggle w-100">
                <label v-for="(property, index) in propertyItem.properties" :key="property.property + index"
                  class="btn btn-primary text-center">
                  <input type="checkbox" :value="property.value"
                    @change.prevent="handlePropertyChanged($event, property)">
                  {{ property.name }}
                </label>
              </div>

              <div v-if="propertyItem.type == propertyTypes.color"
                @change.prevent="handlePropertyChanged($event, propertyItem)">
                <input type="color" class="form-control">
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-if="currentTab == 'workspace'">
        <div class="card mb-2">
          <div class="card-body">
            <label for="">Paper Size</label>
            <select class="form-control" @change="handlePaperSizeChanged($event)">
              <option value="" selected disabled>Select Paper size</option>
              <option v-for="(paperSize, index) in paperSizes" :key="'paperSize-' + index" :value="paperSize.name">
                {{ paperSize.name }}
              </option>
            </select>
          </div>
        </div>

        <div class="card">
          <div class="card-body">
            <label for="">Background</label>
            <label class="cb_file_upload_btn">
              <input type="file" class="form-control" @change="handleBackgroundImageChange" hidden>
              Upload Background Image
            </label>
          </div>
        </div>
      </div>
    </div>
  </div>

</template>

<script>
import PROPERTY_ITEMS_TYPES from '../constants/PropertySidebarItemTypes.js'
export default {
  props: {
    propertySidebarConfig: Object,
    selectedItem: HTMLElement,
    paperSizes: Array,
    currentTab: {
      type: String,
      default() {
        return "workspace";
      }
    }
  },
  data() {
    return {
      propertySidebarConfigData: {},
      propertyTypes: {}
    }
  },
  methods: {
    handlePropertyChanged(e, propertyItem) {
      var newValue = e.target.value;
      console.log({ newValue });
      this.$emit("itemPropertyChanged", {
        selectedItem: this.selectedItem,
        propertyData: propertyItem,
        value: newValue
      });
    },
    handleBackgroundImageChange(e) {
      this.fileToBase64(e.target.files[0])
        .then((data) => {
          this.$emit("backgroundImageChanged", data);
        });
    },
    handlePaperSizeChanged(e) {
      var paperName = e.target.value;
      var paperConfig = this.paperSizes.find(x => x.name == paperName);
      this.$emit("paperSizeChanged", paperConfig);
    },
    fileToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = error => reject(error);
      });
    }
  },
  computed: {
    propertySidebarItems() {
      if (!this.selectedItem) {
        return [];
      }
      return this.propertySidebarConfigData[this.selectedItem.tagName.toLowerCase()];
    }
  },
  created() {
    this.propertyTypes = PROPERTY_ITEMS_TYPES;
    this.propertySidebarConfigData = this.propertySidebarConfig;
  },
  textTick(value) {
    console.log(value);
  }
}
</script>

<style scoped>
*{
  font-size: 12px;
}

.cb_right_sidebar_container {
  min-width: 250px;
  max-width: 250px;
  padding: 16px 16px;
  height: 500px;
}

.card {
  border-radius: 8px;
}

.card-body {
  padding: 8px;
}

.cb_right_sidebar {
  padding: 16px;
  background-color: lightgray;
  border-radius: 12px;
  height: 100%;
}

.cb_property_item label {
  width: 100%;
  text-align: left;
}

.cb_file_upload_btn {
  width: 100%;
  height: 50px;
  border-radius: 25px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: lightgray;
  cursor: pointer;
}

.cb_file_upload_btn:hover {
  background: gray;
}
</style>