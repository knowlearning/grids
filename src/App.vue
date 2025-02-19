<script setup>
  import { ref, computed, reactive } from 'vue'

  const numRows = ref(6)
  const numColumns = ref(6)
  const rowGap = ref(8)
  const columnGap = ref(8)
  const activeArea = ref(null)

  const areas = reactive([])

  const cssGridStyle = computed(() => {
    return `
      display: grid;
      grid-template-columns: repeat(${numColumns.value}, 1fr);
      grid-template-rows: repeat(${numColumns.value}, 1fr);
    `
  })

  const handleDrop = (content, row, column) => {
    areas.push({
      startRow: row,
      startColumn: column,
      endRow: row+1,
      endColumn: column+1,
      content
    })
  }

  window.addEventListener('mouseup', () => {
    if (activeArea.value) {
      if (!areas.includes(activeArea.value)) {
        areas.push(activeArea.value)
      }
      activeArea.value = null
    }
  })
</script>

<template>
  <div id="main">
    <div id="grid"
      :style="`
        user-select: none;
        ${cssGridStyle}
      `"
    >
      <template
        v-for="rowNum in numRows"
        :key="rowNum"
      >
        <div
          class="area-placeholder"
          v-for="columnNum in numColumns"
          :key="columnNum"
          @dragover.prevent
          @drop.prevent="event => {
            handleDrop(
              event.dataTransfer.getData('text/plain'),
              rowNum,
              columnNum
            )
          }"
          @mousedown="
            activeArea = {
              startRow:    rowNum,
              startColumn: columnNum,
              endRow:      rowNum+1,
              endColumn:   columnNum+1
            }"
          @mouseover="() => {
            if (activeArea) {
              activeArea.endRow = rowNum+1
              activeArea.endColumn = columnNum+1
            }
          }"
          :style="`
            border-top: ${rowGap/2}px solid white;
            border-bottom: ${rowGap/2}px solid white;
            border-left: ${columnGap/2}px solid white;
            border-right: ${columnGap/2}px solid white;
            grid-area: ${rowNum} / ${columnNum} / ${rowNum+1} / ${columnNum+1};
          `"
        >
          
        </div>
      </template>
      <div
        v-for="area, index in areas"
        class="area"
        :style="`
          pointer-events: ${ activeArea ? 'none' : 'auto' };
          grid-area: ${area.startRow} / ${area.startColumn} / ${area.endRow} / ${area.endColumn};
        `"
        @mousedown="activeArea = area"
        @dragover.prevent
        @drop.prevent="event => {
          area.content = event.dataTransfer.getData('text/plain')
        }"
      >
        <button
          style="pointer-events: auto;"
          @click="areas.splice(index, 1)"
          @mousedown.stop
        >
          x
        </button>
        {{ area.content }}
      </div>
      <div
        v-if="activeArea"
        class="dragging-area"
        :style="`
          grid-area: ${activeArea.startRow} / ${activeArea.startColumn} / ${activeArea.endRow} / ${activeArea.endColumn};
        `"
      >
      </div>
    </div>
    <div style="width: 256px;">
      rows:
      <input type="number" v-model="numRows" />
      <br>
      columns:
      <input type="number" v-model="numColumns" />
      <br>
      row gap:
      <input type="number" v-model="rowGap" />
      <br>
      column gap:
      <input type="number" v-model="columnGap" />
      <br>
    </div>
  </div>
</template>

<style scoped>
  #main {
    width: 100vw;
    height: 100vh;
    display: flex;
  }

  #grid {
    flex-grow: 1;
  }

  .area {
    background: chartreuse;
    border: 1px solid #888888;
    border-radius: 4px;
    opacity: 0.5;
  }

  .dragging-area {
    background: yellow;
    opacity: 0.5;
    pointer-events: none;
  }

  .area-placeholder {
    background: rgba(0,0,0,0.05);
  }
</style>
