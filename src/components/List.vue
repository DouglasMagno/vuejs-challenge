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
      <Draggable v-for="(list, indexList) in lists" :key="list.id">
        <div class="card-container">
          <div class="card-column-header">
            <span title="Add todo" class="add-item" @click="addTodo(list.id)">&plus;</span>
            <span class="column-drag-handle">&#x2630;</span>
            {{ list.id }} <span title="Remove list" class="remove-item" @click="removeList(list.id)">&times;</span>
          </div>
        </div>
      </Draggable>
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
    /**
     * Add list to lists array if has string length
     */
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
     * Remove list from a list id
     * @param list
     */
    removeList(list) {
      this.$delete(this.lists, this.getListKey(list));
    },

    /**
     * Add todocard method
     * @param list
     */
    addTodo(list) {
      const key = this.getListKey(list);
      this.$set(
          this.lists[key].cards,
          this.lists[key].cards.length,
          {
            name: "",
            id: (this.lists[key].cards.length + 1),
            editing: true
          }
      );

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
     * Get list key from a list id
     * @param listId
    */
    getListKey(listId) {
      const column = this.lists.filter(p => p.id === listId)[0];
      return this.lists.indexOf(column);
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

  .card-container {
    width: 320px;
    margin: 5px;
    margin-right: 45px;
    background-color: #f3f3f3
  }

  .card, .card-container {
    padding: 10px;
    box-shadow: 0 1px 1px rgba(0, 0, 0, .12), 0 1px 1px rgba(0, 0, 0, .24)
  }

  .card-column-header {
    font-size: 18px
  }

  .remove-item {
    cursor: pointer;
    text-align: right;
    float: right;

    &:hover {
      color: black;
    }
  }

  .add-item {
    cursor: pointer;
    text-align: right;
    float: left;

    &:hover {
      color: black;
    }
  }

</style>
