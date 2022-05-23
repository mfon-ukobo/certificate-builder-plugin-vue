<template>
    <div class="cb_left_sidebar_container">
        <div class="cb_left_sidebar">
            <div class="cb_placeholder_widget" v-for="(item, index) in placeholders" :key="index" draggable="true"
                @dragstart="handleStartDrag($event, item)" @dragend="handleEndDrag($event, item)">
                {{ item.name }}
            </div>
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
        },
        handleUploadCertificateImage(e) {
            let $this = this;
            var base64 = this.fileToBase64(e.target.files[0]);
            base64.then((data) => {
                $this.$emit("certificateImageUploaded", data);
            });
        }
    },
    created() {
        this.placeholders = this.placeholdersConfig;
    }
}
</script>

<style scoped>
*{
    font-size: 12px;
}

.cb_left_sidebar_container {
    min-width: 250px;
    max-width: 250px;
    overflow-y: auto;
    padding: 16px;
}

.cb_left_sidebar {
    position: relative;
    background-color: #c6c5c5;
    width: 100%;
    height: 100%;
    padding: 16px 12px;
    border-radius: 24px;
}

.cb_placeholder_widget {
    height: 30px;
    background: white;
    color: black;
    border-radius: 15px;
    box-shadow: 0px 0px 5px -3px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all .2s ease;
    cursor: grab;
    margin-bottom: 8px;
}

.cb_cert_image_upload {
    height: 70px;
    border-radius: 35px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: white;
}

.cb_close_btn {
    border: none;
    position: absolute;
    left: 100%;
    height: 50px;
    width: 50px;
    z-index: +1;
}
</style>