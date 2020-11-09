<template>
    <div ref="container" class="flex-grid">
        <div ref="topBuffer" :style="{ height: topBufferSize + 'px', width: '100%' }"></div>
        <component :is="cellComponent" v-for="item in currentItems" v-bind="{ item: item, itemHeight, columnCount }" :key="item.id"/>
        <div ref="bottomBuffer" :style="{ height: bottomBufferSize + 'px', width: '100%' }"></div>
    </div>
</template>

<script>
    export default {
        name: "VirtualFlexWrap",
        props: {
            items: {
                type: Array,
                required: true
            },
            columnCount: {
                type: Number,
                required: true
            },
            itemHeight: {
                type: Number,
                required: true
            },
            cellComponent: {
                type: Object,
                required: true
            },
            keyProvider: {
                type: Function,
                default: item => item.id
            },
            rowBufferCount: {
                type: Number,
                default: 1
            },
            visibilityCheckInterval: {
                type: Number,
                default: 100
            },
            scrollContainer: {
                type: HTMLDivElement
            }
        },
        data() {
            return {
                observer: null,
                start: 0,
                bufferVisibleIntervals: {
                    top: null,
                    bottom: null
                },
                container: null
            }
        },
        computed: {
            currentItems() {
                return this.items.slice(this.startIndex, this.endIndex);
            },
            startIndex() {
                return Math.max(this.start - this.rowBufferCount * this.columnCount, 0);
            },
            end() {
                return this.start + this.desiredItemCount;
            },
            endIndex() {
                return Math.min(this.end + this.rowBufferCount * this.columnCount, this.items.length);
            },
            topBufferSize() {
                return (this.startIndex / this.columnCount) * this.itemHeight;
            },
            bottomBufferSize() {
                return Math.ceil((this.items.length - this.endIndex) / this.columnCount) * this.itemHeight;
            },
            desiredItemCount() {
                return this.desiredRowCount * this.columnCount;
            },
            displayWindow() { // We technically don't need this, but we want to have joint update for start + end
                return { start: this.startIndex, end: this.endIndex };
            },
            desiredRowCount() {
                return Math.ceil(this.availableHeight / this.itemHeight) + 1;
            },
            usedContainer() {
                if(this.scrollContainer != null) {
                    return this.scrollContainer;
                }
                return this.container;
            },
            availableHeight() {
                return this.usedContainer != null ? this.usedContainer.offsetHeight : 0;
            }
        },
        mounted() {
            const initialStart = this.startIndex;
            const initialEnd = this.endIndex;
            if(initialStart + initialEnd > 0) {
                this.$emit('loaded', [initialStart, initialEnd]);
            }
            this.container = this.$el;
        },
        beforeDestroy() {
            this.detachObserver();
            if(this.bufferVisibleIntervals.top != null) {
                clearInterval(this.bufferVisibleIntervals.top);
                this.bufferVisibleIntervals.top = null;
            }

            if(this.bufferVisibleIntervals.bottom != null) {
                clearInterval(this.bufferVisibleIntervals.bottom);
                this.bufferVisibleIntervals.bottom = null;
            }
        },
        methods: {
            createObserver() {
                const margin = (this.rowBufferCount * this.itemHeight) + "px";
                return new IntersectionObserver(this.onBufferIntersect, {
                    root: this.usedContainer,
                    rootMargin: `${margin} 0px ${margin} 0px`
                });
            },
            refreshObserver() {
                this.detachObserver();
                this.attachObserver();
            },
            attachObserver() {
                this.observer = this.createObserver();
                this.observer.observe(this.$refs.topBuffer);
                this.observer.observe(this.$refs.bottomBuffer);
            },
            detachObserver() {
                if(this.observer) {
                    this.observer.disconnect();
                    this.observer = null;
                }
            },
            onBufferIntersect(entries) {
                const self = this;
                entries.forEach(entry => {
                    const visible = entry.isIntersecting;
                    if(entry.target === self.$refs.topBuffer) {
                        if(visible) {
                            self.loadMoreBefore();
                        }
                    } else if(entry.target === self.$refs.bottomBuffer) {
                        if(visible) {
                            self.loadMoreAfter();
                        }
                    }
                });
            },
            loadMoreBefore(rows = 1) {
                if(this.start <= 0) {
                    return;
                }

                this.start -= rows * this.columnCount;
                this.startBufferCheckBefore();
            },
            loadMoreAfter(rows = 1) {
                if(this.end >= this.items.length) {
                    return;
                }

                this.start += rows * this.columnCount;
                this.startBufferCheckAfter();
            },
            startBufferCheckBefore() {
                if(this.bufferVisibleIntervals.top != null) {
                    return;
                }

                this.bufferVisibleIntervals.top = setInterval(() => {
                    const changed = this.adjustToMatchScroll();

                    if(!changed) {
                        clearInterval(this.bufferVisibleIntervals.top);
                        this.bufferVisibleIntervals.top = null;
                    }
                }, this.visibilityCheckInterval);
            },
            startBufferCheckAfter() {
                if(this.bufferVisibleIntervals.bottom != null) {
                    return;
                }

                this.bufferVisibleIntervals.bottom = setInterval(() => {
                    const changed = this.adjustToMatchScroll();

                    if(!changed) {
                        clearInterval(this.bufferVisibleIntervals.bottom);
                        this.bufferVisibleIntervals.bottom = null;
                    }
                }, this.visibilityCheckInterval);
            },
            adjustToMatchScroll() {
                const scrollPosition = this.usedContainer.scrollTop;
                const rowPosition = scrollPosition / this.itemHeight;
                const previousStart = this.start;
                const previousEnd = this.end;
                let newStart = Math.floor(rowPosition) * this.columnCount;
                let newEnd = newStart + this.desiredItemCount;

                if(newStart === previousStart && newEnd === previousEnd) {
                    return false;
                }

                this.start = newStart;
                return true;
            }
        },
        watch: {
            items(newValue, oldValue) {
                if(oldValue.length === newValue.length) {
                    return;
                }

                const newItemCount = newValue.length;
                if(newItemCount < this.desiredItemCount) {
                    // if item count is smaller than the desired -> not enough space aka we don't need buffers
                    this.start = 0;
                }

                if(newItemCount > oldValue.length) {
                    this.startBufferCheckAfter();
                }
            },
            columnCount(newValue, oldValue) {
                if(newValue === oldValue) {
                    return;
                }

                this.start = (this.start / oldValue) * newValue;
            },
            itemHeight(newValue, oldHeight) {
                const currentScrollRow = this.usedContainer.scrollTop / oldHeight;
                this.usedContainer.scrollTop = currentScrollRow * newValue;
            },
            displayWindow(newWindow, oldWindow) {
                if(newWindow.start === oldWindow.start && newWindow.end === oldWindow.end) {
                    return;
                }

                if(newWindow.end < oldWindow.start || newWindow.start > oldWindow.end) {
                    // no overlap at all
                    this.$emit('unloaded', [oldWindow.start, oldWindow.end]);
                    this.$emit('loaded', [newWindow.start, newWindow.end]);
                    return;
                }

                if(newWindow.start > oldWindow.start) {
                    this.$emit('unloaded', [oldWindow.start, newWindow.start - 1]);
                } else if(newWindow.start < oldWindow.start) {
                    this.$emit('loaded', [newWindow.start, oldWindow.start - 1]);
                }

                if(newWindow.end > oldWindow.end) {
                    let start;
                    if(oldWindow.end === oldWindow.start) {
                        start = 0;
                    } else {
                        start = oldWindow.end + 1;
                    }
                    this.$emit('loaded', [start, newWindow.end]);
                } else if(newWindow.end < oldWindow.end) {
                    this.$emit('unloaded', [newWindow.end + 1, oldWindow.end]);
                }
            },
            usedContainer() {
                this.refreshObserver();
            }
        }
    }
</script>

<style scoped>
    .flex-grid {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
        overflow-y: auto;
        align-content: flex-start;
        height: 100%;
    }
</style>
