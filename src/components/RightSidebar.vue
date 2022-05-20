<template>
  <div class="cb_right_sidebar">
    <div v-for="(item, index) in propertySidebarItems" :key="index">
      <div class="card card-body mb-2">
        <label>{{item.name}}</label>
        
        <div v-if="item.type == propertyTypes.dropdown">
          <select class="form-control" @change.prevent="handlePropertyChanged">
            <option :value="option.value" v-for="(option, index2) in item.options" :key="item.property + index2">
              {{option.name}}
            </option>
          </select>
        </div>

        <div v-if="item.type == propertyTypes.checkbox" class="btn-group btn-group-toggle">
          <label v-for="(property, index) in item.properties" :key="property.property + index" class="btn btn-primary">
            <input type="checkbox" @change.prevent="handlePropertyChanged">
            {{property.name}}
          </label>
        </div>

        <div v-if="item.type == propertyTypes.color" @change.prevent="handlePropertyChanged">
          <input type="color" class="form-control">
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
  },
  data() {
    return{
      propertySidebarConfigData: {},
      propertyTypes: {}
    }
  },
  methods: {
    handlePropertyChanged() {
      alert();
      this.$emit("propertyChanged", this.selected)
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
  }
}
</script>

<style scoped>
.cb_right_sidebar {
  background-color: red;
  min-width: 250px;
  padding: 4px;
}
</style>