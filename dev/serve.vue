<script>
    import VirtualFlexWrap from '@/virtual-flex-wrap.vue';
    import Vue from 'vue';
    import ExampleElement from "./example-element.vue";

    function getRandomColor() {
        const letters = '0123456789ABCDEF';
        let color = '#';
        for(let i = 0; i < 6; i++) {
            color += letters[Math.floor(Math.random() * 16)];
        }
        return color;
    }

    function generateElements(amount) {
        const array = [];

        for(let i = 0; i < amount; i++) {
            array.push({
                id: i + 1,
                color: getRandomColor()
            });
        }

        return array;
    }

    export default Vue.extend({
        name: 'ServeDev',
        components: {
            VirtualFlexWrap
        },
        data() {
            return {
                component: ExampleElement,
                buffer: 1,
                columns: 4,
                height: 400,
                elements: generateElements(100)
            }
        },
        methods: {
            addItems() {
                const offset = this.elements.length;
                for(let i = 0; i < 4; i++) {
                    this.elements.push({
                        id: i + offset + 1,
                        color: getRandomColor()
                    })
                }
            },
            loadedRange(range) {
                console.log('Loaded: ', range);
            },
            unloadedRange(range) {
                console.log('Unloaded: ', range);
            }
        }
    });
</script>

<template>
    <div id="app">
        <button @click="buffer += 1">Increase buffer</button>
        <button @click="buffer -= 1">Decrease buffer</button>
        Buffer: {{ buffer }}
        <button @click="addItems">Add Items</button>
        <button @click="columns += 1">Add Column</button>
        <button @click="columns -= 1">Remove Column</button>
        <input type="range" min="100" max="1000" step="50" v-model.number="height"/>{{ height }}
        <div style="height: 90%">
            <VirtualFlexWrap :cell-component="component" :items="elements" :item-height="height" :column-count="columns"
                             :row-buffer-count="buffer" @loaded="loadedRange" @unloaded="unloadedRange"/>
        </div>
    </div>
</template>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        height: 95vh;
    }

    body {
        background-color: grey;
    }
</style>
