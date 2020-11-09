<template>
    <div :style="{ backgroundColor: item.color, height: itemHeight + 'px', width: (100 / columnCount) + '%' }">
        Testing {{item.id}}
    </div>
</template>

<script>
    export default {
        name: "ExampleElement",
        props: ['item', 'columnCount', 'itemHeight']
    }
</script>

<style scoped>

</style>
