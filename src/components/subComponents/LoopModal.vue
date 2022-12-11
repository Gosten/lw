<script>
module.exports = {
  data() {
    return {
      loopState: {
        order: [],
        selectedGrip: ""
      }
    };
  },
  methods: {
    handleGripClick(id) {
      isGripSelected = Boolean(this.problemState[id]);
      if (!isGripSelected) return;

      this.loopState.selectedGrip = id;
      if (!this.loopState.order.includes(id)) {
        this.loopState.order.push(id);
      }
    },
    handleSubmit() {
      this.setLoopState(this.loopState.order);
      this.close();
    }
  },
  props: {
    setLoopState: Function,
    close: Function,
    initialOrder: Array
  },
  mounted() {
    const initialOrder = this.initialOrder || [];
    this.loopState.order = initialOrder;
  },
  components: {
    board: httpVueLoader("components/Board/Board.vue"),
    "loop-number-display": httpVueLoader(
      "components/subComponents/LoopGripNumbering.vue"
    )
  },
  computed: {
    problemState() {
      return this.$store.getters.getAddProblemState;
    }
  }
};
</script>

<style scoped type="text/css">
.add-problem-modal {
  height: calc(var(--window-height) - 2em - 60px);
  width: 100%;
  margin-block: calc(1em + 60px) 1em;
  margin-inline: 1em;
  padding: 1em;
  flex-direction: column;
}
.board-position-loop-setup {
  width: 100%;
  height: 100%;
}
.loop-modal {
  display: grid;
  grid-template-columns: 100%;
  grid-template-rows: auto 60% 4em auto;
}
</style>

<template>
  <div id="popup-container" class="popup-container">
    <div class="popup-window add-problem-modal loop-modal">
      <p>Numeracja chwytów</p>
      <div id="board-style-AP" class="board-position-loop-setup">
        <board
          board-id="board-loop-setup"
          alt-pinch-id="loop-pinch"
          :grip-click-callback="handleGripClick"
          :loop-state="loopState"
        ></board>
      </div>
      <loop-number-display
        :loop-state="loopState"
        :set-selected-grip="handleGripClick"
      ></loop-number-display>
      <button class="button" @click="handleSubmit">Zatwierdź</button>
      <button class="close-button" @click="close">
        <i class="mdi mdi-close"></i>
      </button>
    </div>
  </div>
</template>
