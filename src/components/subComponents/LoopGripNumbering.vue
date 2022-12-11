<script>
module.exports = {
  data() {
    return {
      mainContainerHandle: null,
      loopNumberItemInView: null
    };
  },
  methods: {
    scrollToSelected(id) {
      const selectedElementHandle = document.getElementById(
        `loop-number-item-${id}`
      );
      if (selectedElementHandle && id !== this.loopNumberItemInView) {
        selectedElementHandle.scrollIntoView({ behavior: "smooth" });
        this.loopNumberItemInView = id;
      }
    }
  },
  computed: {},
  props: {
    loopState: Object,
    setSelectedGrip: Function
  },
  watch: {
    "loopState.selectedGrip": {
      handler(newSelectedGripId, o) {
        this.scrollToSelected(newSelectedGripId);
      },
      deep: true
    }
  },
  updated() {
    this.scrollToSelected(this.loopState.selectedGrip);
  },
  components: {},
  mounted() {
    this.mainContainerHandle = document.getElementById(
      "loop-numbering-display"
    );
  }
};
</script>

<style scoped type="text/css">
.loop-grip-numbering-container {
  width: 100%;
  padding-bottom: 10px;
  position: relative;
  display: flex;
  align-items: center;
  /* justify-content: center; */
  overflow-x: scroll;
}
.loop-numbers-wrapper-container {
  /* position: absolute; */
  display: flex;
  height: 4em;
  width: fit-content;
}
.loop-number-item {
  font-size: 2em;
  display: flex;
  /* padding: 0.2em 0.35em; */
  width: 2em;
  justify-content: center;
  align-items: center;
  transition-duration: 0.15s ease-in-out;
  box-sizing: border-box;
  margin: 2px;
  border: 1px solid grey;
  border-radius: 3px;
}
.selected-number {
  border: 2px solid rgb(0, 47, 255);
  /* font-size: 3em; */
}
</style>

<template>
  <div class="loop-grip-numbering-container" id="loop-numbering-display">
    <div class="loop-numbers-wrapper-container">
      <div
        class="loop-number-item"
        v-for="(id, index) in loopState.order"
        :key="`${id}-${index}`"
        :class="{ 'selected-number': id === loopState.selectedGrip }"
        :id="`loop-number-item-${id}`"
        @click="() => setSelectedGrip(id)"
      >
        {{ index }}
      </div>
    </div>
  </div>
</template>
