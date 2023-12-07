<template>
    <div class="calc-display">
        <div id="menu-icon-place" class="storage-window">menuicon</div>
        <div class="calc-current">{{ calcDisp }}</div>
        <div class="calc-mem">calcmem</div>
        <div><span class="save-button saveIcon" id="save-icon-place">saveIco</span></div>
    </div>
</template>

<script>
export default {
    props: {
        calcDisp: String
    },
};
</script>