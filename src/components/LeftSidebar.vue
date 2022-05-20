<template>
<div class="cb_left_sidebar">
    <div class="cb_placeholder_widget" 
        v-for="(item, index) in placeholders" 
        :key="index"
        draggable="true"
        @dragstart="handleStartDrag($event, item)"
        @dragend="handleEndDrag($event, item)">
        {{item.name}}
    </div>
</div>  
</template>

<script>
export default {
    props: {
        placeholdersConfig: Array
    },
    data() {
        return {
            placeholders: []
        }
    },
    methods: {
        handleStartDrag(e, widget) {
            e.target.style.opacity = '0.5';

            e.dataTransfer.setData('widget', widget.id);
        },
        handleEndDrag(e) {
            e.target.style.opacity = '1';
        }
    },
    created() {
        this.placeholders = this.placeholdersConfig;
    }
}
</script>

<style scoped>
.cb_left_sidebar {
    min-width: 250px;
    background-color: red;
    padding: 16px;
}

.cb_placeholder_widget {
    height: 30px;
    background: white;
    border-radius: 4px;
    box-shadow: 0px 0px 5px -3px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all .2s ease;
    cursor: grab;
    margin-bottom: 8px;
}
</style>