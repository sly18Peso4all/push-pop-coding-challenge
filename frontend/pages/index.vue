<template>
  <div class="error-handler">
    <div class="resolved">
      <h3>Resolved Errors:</h3>
      <div v-for="error in resolved" :key="error.index" class="error-box green">
        <h5>Error-Code: {{ error.code }} - {{ error.text }}</h5>
        <button type="button" @click="resolveErrorData(error)">
          Unresolve Error
        </button>
      </div>
    </div>
    <div class="unresolved">
      <h3>Unresolved Errors:</h3>
      <div v-for="error in unresolved" :key="error.index" class="error-box red">
        <h5>
          Error-Code:
          {{ error.code }} - {{ error.text }}
        </h5>
        <button type="button" @click="unResolveErrorData(error)">
          Resolve Error
        </button>
      </div>
    </div>
    <div class="backlog">
      <h3>Backlog:</h3>
      <div v-for="error in backlog" :key="error.index" class="error-box blue">
        <h5>Error-Code: {{ error.code }} - {{ error.text }}</h5>
        <button type="button" @click="moveBacklogErrorData(error)">
          Move Error
        </button>
      </div>
    </div>
    <div class="button-view">
      <button type="button" class="undo-button m3" @click="undoLastAction">
        Undo-error
      </button>
      <button type="button" class="undo-button teal m3" @click="undoAllActions">
        Undoall
      </button>
    </div>
  </div>
</template>

<script>
let undoActions = [];

export default {
  async asyncData({ $axios }) {
    try {
      const { resolved, unresolved, backlog } = await $axios.$get(
        "http://localhost:8000/get_lists"
      );
      return {
        resolved,
        unresolved,
        backlog
      };
    } catch (error) {
      console.log(
        `Couldn't get error lists:\n${error}\nDid you start the API?`
      );
      console.log(
        "HINT: You can comment out the full `asyncData` method and work with mocked data for UI/UX development, if you want to."
      );
    }
  },
  data() {
    return {
      resolved: [],
      unresolved: [],
      backlog: [],
      undoActions: undoActions
    };
  },

  methods: {
    resolveErrorData(unresolvedError) {
      const index = this.resolved.findIndex(
        item => item.index == unresolvedError.index
      );

      unresolvedError.text = unresolvedError.text.replace(
        "resolved",
        "unresolved"
      );
      this.resolved.splice(index, 1);

      this.unresolved.push(unresolvedError);
      this.undoActions.push({
        index: unresolvedError.index,
        from: this.unresolved,
        to: this.resolved
      });
      return;
    },

    unResolveErrorData(resolvedError) {
      const index = this.unresolved.findIndex(
        item => item.index == resolvedError.index
      );
      console.log("helloi" + index);
      resolvedError.text = resolvedError.text.replace("unresolved", "resolved");
      this.unresolved.splice(index, 1);
      this.resolved.push(resolvedError);
      this.undoActions.push({
        index: resolvedError.index,
        from: this.resolved,
        to: this.unresolved
      });

      return;
    },
    moveBacklogErrorData(backlogError) {
      const index = this.backlog.findIndex(
        item => item.index == backlogError.index
      );
      console.log(index);
      backlogError.text = backlogError.text.replace("backlog", "unresolved");
      this.backlog.splice(index, 1);
      this.unresolved.push(backlogError);
      this.undoActions.push({
        index: backlogError.index,
        from: this.unresolved,
        to: this.backlog
      });

      return;
    },
    undoLastAction() {
      if (undoActions.length != 0) {
        this.undo();
      }
    },
    undoAllActions() {
      let lengthOfUndoActions = undoActions.length;
      for (let i = 0; i < lengthOfUndoActions; i++) {
        this.undo();
        console.log(i);
      }
    },
    undo() {
      let lastAction = undoActions.pop();
      const index = lastAction.from.findIndex(
        item => item.index == lastAction.index
      );
      let error = lastAction.from.splice(index, 1);
      lastAction.to.push(error.pop());
      console.log(lastAction);
    }
  }
};
</script>

<style scoped>
.error-handler {
  display: flex;
  justify-content: space-around;
  border: 1px solid black;
  padding: 3rem;
  background: black;
  color: #f1f1f1;
}

button {
  border: none;
  padding: 10px;
  background: rgb(19, 18, 17);
  color: #f1f1f1;
  border: 1px solid transparent;
  outline: none;
  transition: all 0.4s;
}

button:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
}

h5 {
  margin-bottom: 1rem;
}

button:active {
  transform: translateY(-1px);
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2);
}

.undo-button {
  display: flex;
  align-items: flex-start;
  background: orange;
}

.error-box {
  border: 1px solid transparent;
  padding: 1.5rem;
  margin: 10px 0;
  color: #f1f1f1;
}

.m3 {
  margin-right: 10px;
}

.button-view {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
  margin-top: 2rem;
}

.teal {
  background: teal;
}

.green {
  background: green;
}

.red {
  background: rgb(224, 59, 59);
}

.blue {
  background: rgb(87, 87, 211);
}
</style>
