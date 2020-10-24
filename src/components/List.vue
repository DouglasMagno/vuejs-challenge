<template>
  <div>
    <input type="text" v-model="newList" class="todo-input" placeholder="Add a new list and press enter"
           @keyup.enter="addList">

    <Container
      orientation="horizontal"
      @drop="onColumnDrop($event)"
      drag-handle-selector=".column-drag-handle"
      @drag-start="dragStart"
      class="wrapper-two-column"
    >
    </Container>

  </div>
</template>

<script>
import {Container, Draggable} from 'vue-smooth-dnd';
export default {
  name: "List",
  components: {
    Container,
    Draggable
  },
  data() {
    return {
      newList: '',
      lists: [
        {
          id: "First List",
          cards: [],
        }
      ],
    };
  },
  methods: {
    addList() {
      if (this.newList.trim().length === ) {
        return;
      }

      this.$set(
          this.lists,
          this.lists.length,
          {
            id: this.newList,
            cards: [],
          }
      );
      this.newList = '';
    },
    /**
     * Event handler when drop a column, perform and change positions on list array
     * @param dropResult
     */
    onColumnDrop(dropResult) {
      this.$set(this, "lists", this.applyDrag(this.lists, dropResult));
    },

    /**
     * this method it will run when start a drag
     */
    dragStart() {

    },

    /**
     * Add or remove list or card when move
     * @param arr
     * @param dragResult
     * @returns {*[]|*}
     */
    applyDrag(arr, dragResult) {
      // get vars to remove or add with you payload
      const {removedIndex, addedIndex, payload} = dragResult
      if (removedIndex === null && addedIndex === null) return arr

      // interpolation the result
      const result = [...arr]
      let itemToAdd = payload

      // case need remove index of array
      if (removedIndex !== null) {
        itemToAdd = result.splice(removedIndex, 1)[0]
      }

      // case need add index on array
      if (addedIndex !== null) {
        result.splice(addedIndex, 0, itemToAdd)
      }

      return result
    }
  }
}
</script>

<style lang="scss">
  .todo-input {
    width: 100%;
    padding: 10px 18px;
    font-size: 18px;
    margin-bottom: 16px;

    &:focus {
      outline: 0;
    }
  }
  .column-drag-handle {
    cursor: move;
    padding: 5px
  }

  .wrapper-two-columns {
    display: grid !important;
    grid-template-columns: 48% 49%;
    grid-gap: 10px;
  }
</style>
