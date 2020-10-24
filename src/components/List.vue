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
          <Container
            group-name="col"
            @drop="(e) => onCardDrop(list.id, e)"
            @drag-start="(e) => myLog('drag start', e)"
            @drag-end="(e) => myLog('drag end', e)"
            :get-child-payload="getCardPayload(list.id)"
            drag-class="card-ghost"
            drop-class="card-ghost-drop"
          >
            <Draggable v-if="list.cards.length" v-for="(card, index) in list.cards" :key="card.id">
              <div v-if="!card.editing" @dblclick="editTodo(indexList, index)" class="card">{{ card.id }} - {{ card.name }}
                <div title="Remove todo" class="remove-item" @click="removeTodo(indexList, index)">&times;</div>
              </div>
              <input v-else class="todo-item-edit" type="text" v-model="card.name"
                     @blur="doneEdit(indexList, index, card)" @keyup.enter="doneEdit(indexList, index, card)"
                     @keyup.esc="cancelEdit(indexList, index)" v-focus>
            </Draggable>
          </Container>
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
      beforeEditCache: '',
      tempTable: null,
      tempColumn: null,
      lists: [
        {
          id: "First List",
          cards: [],
        },
      ],
    };
  },
  directives: {
    focus: {
      inserted: function (el) {
        el.focus();
      }
    }
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
     * Open todocard to edit
     * @param list
     * @param index
     */
    editTodo(list, index) {
      this.beforeEditCache = this.lists[list].cards[index].name;
      this.$set(this.lists[list].cards[index], "editing", true);
    },

    /**
     * Finish a edition on a card
     * @param list
     * @param index
     * @param todo
     */
    doneEdit(list, index, todo) {
      if (todo.name.trim().length === 0) {
        this.$set(this.lists[list].cards[index], "name", this.beforeEditCache);
      }
      this.$set(this.lists[list].cards[index], "editing", false);
    },

    /**
     * Cancel an edition
     * @param list
     * @param index
     * @param todo
     */
    cancelEdit(list, index, todo) {
      this.$set(this.lists[list].cards[index], "name", this.beforeEditCache);
      this.$set(this.lists[list].cards[index], "editing", false);
    },

    /**
     * Remove a todocard on a list
     * @param list
     * @param index
     * @param todo
     */
    removeTodo(list, index, todo) {
      this.$delete(this.lists[list].cards, index);
    },

    /**
     * Event handler when drop a column, perform and change positions on list array
     * @param dropResult
     */
    onColumnDrop(dropResult) {
      this.$set(this, "lists", this.applyDrag(this.lists, dropResult));
    },

    /**
     * Handler to change cards when they are dropped
     * @param columnId
     * @param dropResult
     */
    onCardDrop(columnId, dropResult) {
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        const column = this.lists.filter(p => p.id === columnId)[0]
        const columnIndex = this.lists.indexOf(column)
        const newColumn = Object.assign({}, column)
        newColumn.cards = this.applyDrag(newColumn.cards, dropResult)
        this.lists.splice(columnIndex, 1, newColumn)
      }
    },

    /**
     * Get card payload on drag to perform mutation
     * @param columnId
     */
    getCardPayload(columnId) {
      return index => {
        return this.lists.filter(p => p.id === columnId)[0].cards[index]
      }
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
     * My log handler to debug
     */
    myLog(...log) {

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

  .card {
    margin: 5px;
    background-color: #fff
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

  .card-ghost {
    transition: transform .18s ease;
    transform: rotate(5deg)
  }

  .card-ghost-drop {
    transition: transform .18s ease-in-out;
    transform: rotate(0deg)
  }

  .todo-item-edit {
    font-size: 24px;
    color: #2c3e50;
    margin-left: 12px;
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;

    &:focus {
      outline: none;
    }
  }

</style>
