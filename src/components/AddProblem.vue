<template>
  <div id="add-problem" class="grid-content">
    <div id="add-problem-top" class="top-conainer">
      <div id="board-style-AP" class="board-position">
        <transition name="board-fade">
          <board board-id="board-AdP"></board>
        </transition>
      </div>

      <board-legend></board-legend>
      <scroll-arrow scroll-container-id="add-problem"></scroll-arrow>
    </div>

    <div class="flex-container-blank" id="add-bottom">
      <div class="bottom">
        <scroll-arrow
          scroll-container-id="add-problem"
          :reverse="true"
        ></scroll-arrow>
        <div class="input-container">
          <div class="name-grade-container">
            <name-input
              :set-name="setName"
              :set-author="setAuthor"
              :error-name="errorFlags.name"
              :set-error-name="setErrorFlag"
            ></name-input>
            <div v-if="ENABLE_GRADES" class="slider-width">
              <grade-slider></grade-slider>
            </div>
          </div>
          <comment-field :set-comment="setComment"></comment-field>
        </div>
        <div class="button-container">
          <button class="button" @click="addProblem">Dodaj problem</button>
        </div>
      </div>
    </div>

    <transition name="fade">
      <error-modal v-if="errorState.active"></error-modal>
    </transition>

    <transition name="fade">
      <problem-modal v-if="addModal"></problem-modal>
    </transition>
  </div>
</template>

<script>
module.exports = {
  mounted() {
    this.$store.commit("clearProblemState", gripMap);
  },
  beforeUnmount() {
    this.textInputHandle.removeEventListener("keyup", this.blurInput);
  },
  data() {
    return {
      BOARD_CONFIG,
      nameValue: "",
      authNameValue: "",
      commentValue: "",
      errorFlags: {
        name: false
      },
      mapGrade,
      boardZoom: false,
      ENABLE_ZOOM,
      ENABLE_GRADES,
      ENABLE_AUTHOR,
      ADD_PROBLEM,
      textInputHandle: undefined
    };
  },
  components: {
    board: httpVueLoader("components/Board/Board.vue"),
    "name-input": httpVueLoader("components/subComponents/NameAuthInput.vue"),
    "error-modal": httpVueLoader("components/ErrorModal.vue"),
    "problem-modal": httpVueLoader("components/AddProblemModal.vue"),
    "loading-modal": httpVueLoader("components/LoadingModal.vue"),
    "grade-slider": httpVueLoader("components/SingleSlider.vue"),
    "board-legend": httpVueLoader("components/subComponents/BoardLegend.vue"),
    "scroll-arrow": httpVueLoader("components/subComponents/ScrollArrow.vue"),
    "comment-field": httpVueLoader("components/subComponents/CommentField.vue")
  },
  computed: {
    problmState() {
      return this.$store.getters.getAddProblemState;
    },
    errorState() {
      return this.$store.getters.getErrorState;
    },
    problemList() {
      return [...this.$store.getters.getProblemList];
    },
    addModal() {
      return this.$store.getters.getAddProblemModal;
    },
    gradeValue() {
      return this.$store.getters.getSelectedGrade;
    },
    nameLengthLimit() {
      return this.nameValue.length >= 20;
    },
    textInputFocus() {
      return this.$store.getters.getTextInputFocus;
    },
    zoom() {
      return this.$store.getters.getZoom(ADD_PROBLEM);
    },
    zoomScale() {
      return this.$store.getters.getZoomScale(ADD_PROBLEM);
    }
  },
  methods: {
    setName(newName) {
      this.nameValue = newName;
    },
    setAuthor(newAuthor) {
      this.authNameValue = newAuthor;
    },
    setComment(newComment) {
      this.commentValue = newComment;
    },
    setErrorFlag(newState) {
      this.errorFlags.name = newState;
    },
    handleChange(value) {
      this.gradeValue = value;
    },

    blurInput(e) {
      if (e.key == "Enter") this.textInputHandle.blur();
    },

    addProblem() {
      let newProblem = {
        name: this.nameValue,
        grade: mapGrade(this.gradeValue),
        grips: this.problmState,
        author: this.authNameValue,
        comment: this.commentValue || "",
        timestamp: Date.now()
      };
      if (this.validateProblem(newProblem)) {
        let newList = [newProblem, ...this.problemList];

        uibuilder.send({
          topic: "save",
          payload: JSON.stringify(newList)
        });
        uibuilder.send({ topic: TOPIC.LOAD });

        this.$store.commit("toggleAddProblemModal");
        this.$store.commit("setAddProblemNewProblem", newProblem);
      }
    },

    validateProblem({ name, grips }) {
      //validate name
      if (!name) return this.handleNameError("Wpisz nazwę problemu");
      if (name.length > 20) return;

      //validate grips
      let selecterGripsAmount = Object.keys(grips).filter(
        (key) => grips[key]
      ).length;
      if (selecterGripsAmount === 0)
        return this.handleError("Nie wybrano żadnego chwytu");

      //validate name uniqueness
      return this.checkNameUniqueness(name);
    },

    checkNameUniqueness(name) {
      let matchArray = this.problemList.filter(
        (problem) => problem.name === name
      );
      if (matchArray.length === 0) return true;
      return this.handleNameError("Ta nazwa już istnieje");
    },

    handleError(errorMessage) {
      this.$store.dispatch("toggleErrorModal", {
        active: true,
        message: errorMessage
      });
      console.warn(errorMessage);
      return false;
    },

    handleNameError(errorMessage) {
      this.handleError(errorMessage);
      this.errorFlags.name = true;
    }
  }
};
</script>

<style scoped type="text/css">
@import url("../css/AddProblem.css");

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

.border-red {
  border-color: red !important;
}
.name-input-container > p {
  margin: 0 5px;
  color: red;
  font-size: 0.7em;
}
.name-input-container > input {
  margin-bottom: 0 !important;
}
.opacity {
  opacity: 0;
}
</style>
