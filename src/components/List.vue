<template>
  <div>
    <input v-model="newList" type="text" class="todo-input" placeholder="Add a new list and press enter"
           @keyup.enter="addList">

    <Container
        orientation="horizontal"
        @drop="onColumnDrop($event)"
        drag-handle-selector=".column-drag-handle"
        @drag-start="dragStart"
        class="wrapper-two-columns"
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
            <Draggable v-for="(card, index) in list.cards" :key="card.id">
              <div v-if="!card.editing" @dblclick="editTodo(indexList, index)" class="card"
                   :class="{completed : card.completed}">{{ card.id }} - {{ card.name }}
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
      if (this.newList.trim().length === 0) {
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
     */
    cancelEdit(list, index) {
      this.$set(this.lists[list].cards[index], "name", this.beforeEditCache);
      this.$set(this.lists[list].cards[index], "editing", false);
    },

    /**
     * Remove a todocard on a list
     * @param list
     * @param index
     */
    removeTodo(list, index) {
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
    myLog() {

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

.wrapper-two-columns {
  display: grid !important;
  grid-template-columns: 48% 49%;
  grid-gap: 10px;
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

.header {
  height: 60px;
  background-color: #41b883;
  box-shadow: 2px 0 4px #ccc;
  z-index: 0;
  color: #fff;
  font-size: 20px;
  padding: 0 50px;
  vertical-align: middle;
  line-height: 60px
}

.header.open {
  padding: 0 20px
}

.source-code {
  float: right;
  height: 60px;
  line-height: 60px;
  vertical-align: middle;
  cursor: pointer
}

.source-code img {
  width: 30px;
  height: 30px;
  vertical-align: middle
}

.source-code span {
  font-size: 14px;
  color: #eee;
  margin-left: 10px;
  vertical-align: middle
}

.nav-button {
  display: none;
  position: fixed;
  left: 10px;
  top: 20px;
  z-index: 3;
  width: 30px;
  height: 30px;
  -webkit-transform: rotate(0deg);
  -moz-transform: rotate(0deg);
  -o-transform: rotate(0deg);
  transform: rotate(0deg);
  -webkit-transition: .3s ease-in-out;
  -moz-transition: .3s ease-in-out;
  -o-transition: .3s ease-in-out;
  transition: .3s ease-in-out;
  cursor: pointer
}

.nav-button.open {
  left: 310px
}

.nav-button.open span {
  background: #ccc
}

.nav-button span {
  display: block;
  position: absolute;
  height: 4px;
  width: 100%;
  background: #fff;
  border-radius: 4px;
  opacity: 1;
  left: 0;
  -webkit-transform: rotate(0deg);
  -moz-transform: rotate(0deg);
  -o-transform: rotate(0deg);
  transform: rotate(0deg);
  -webkit-transition: .25s ease-in-out;
  -moz-transition: .25s ease-in-out;
  -o-transition: .25s ease-in-out;
  transition: .25s ease-in-out
}

.nav-button span:first-child {
  top: 0
}

.nav-button span:nth-child(2), .nav-button span:nth-child(3) {
  top: 8px
}

.nav-button span:nth-child(4) {
  top: 16px
}

.nav-button.open span:first-child {
  top: 18px;
  width: 0;
  left: 50%
}

.nav-button.open span:nth-child(2) {
  -webkit-transform: rotate(45deg);
  -moz-transform: rotate(45deg);
  -o-transform: rotate(45deg);
  transform: rotate(45deg)
}

.nav-button.open span:nth-child(3) {
  -webkit-transform: rotate(-45deg);
  -moz-transform: rotate(-45deg);
  -o-transform: rotate(-45deg);
  transform: rotate(-45deg)
}

.nav-button.open span:nth-child(4) {
  top: 8px;
  width: 0;
  left: 50%
}

.navigator {
  width: 350px;
  flex-shrink: 0;
  box-shadow: 0 2px 5px #ccc;
  background-color: #fff;
  z-index: 1;
  transition: width .3s ease-in-out;
  overflow: hidden
}

@media (max-width: 700px) {
  .navigator {
    position: fixed;
    height: 100%;
    left: 0;
    top: 0;
    bottom: 0
  }
  .nav-button {
    display: block
  }
}

.nav-panel {
  flex-shrink: 0;
  width: 300px
}

.nav.closed {
  width: 0
}

.nav-content {
  padding-top: 0;
  width: 350px;
  height: 100%;
  overflow-y: auto
}

.nav-header h3 {
  padding-left: 10px
}

.divider {
  height: 1px;
  border-bottom: 1px solid #ddd;
  margin-top: -2px
}

.menu-section h4 {
  color: #444;
  padding-left: 10px
}

.menu-section ul {
  color: #666;
  list-style: none;
  padding: 0;
  margin: 0
}

.menu-section li {
  font-size: 14px;
  margin: 0;
  padding: .5em .5em .5em 30px;
  cursor: pointer;
  margin-bottom: 5px
}

.menu-section li.selected, .menu-section li:hover, .router-link-active {
  background-color: #f8f8ff
}

.draggable-item {
  height: 50px;
  line-height: 50px;
  margin-bottom: 2px;
  margin-top: 2px;
  user-select: none
}

.draggable-item, .draggable-item-horizontal {
  text-align: center;
  display: block;
  background-color: #fff;
  outline: 0;
  border: 1px solid rgba(0, 0, 0, .125);
  cursor: default
}

.draggable-item-horizontal {
  height: 300px;
  padding: 10px;
  line-height: 100px;
  margin-right: 4px
}

.dragging {
  background-color: #ff0
}

.card-scene {
  padding: 50px
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

.card {
  margin: 5px;
  background-color: #fff
}

.card-column-header {
  font-size: 18px
}

.column-drag-handle {
  cursor: move;
  padding: 5px
}

.card-ghost {
  transition: transform .18s ease;
  transform: rotate(5deg)
}

.card-ghost-drop {
  transition: transform .18s ease-in-out;
  transform: rotate(0deg)
}

.opacity-ghost {
  transition: all .18s ease;
  opacity: .8;
  background-color: #6495ed;
  box-shadow: 3px 3px 10px 3px rgba(0, 0, 0, .3)
}

.opacity-ghost-drop {
  opacity: 1;
  background-color: #fff;
  box-shadow: 3px 3px 10px 3px transparent
}

.form-demo {
  width: 650px;
  margin-left: auto;
  margin-right: auto;
  margin-top: 50px;
  display: flex
}

.form {
  flex: 3;
  border: 1px solid rgba(0, 0, 0, .125);
  border-radius: 6px;
  box-shadow: 3px 3px 3px rgba(0, 0, 0, .08), 0 3px 3px rgba(0, 0, 0, .08)
}

.form-fields-panel {
  flex: 1;
  margin-right: 50px
}

.form-ghost {
  transition: .18s ease;
  box-shadow: 1px 1px 5px 2px rgba(0, 0, 0, .08)
}

.form-ghost-drop {
  box-shadow: 0 0 2px 5px transparent
}

.drop-preview {
  margin: 5px
}

.cards-drop-preview, .drop-preview {
  background-color: rgba(150, 150, 200, .1);
  border: 1px dashed #abc
}

.cards-drop-preview {
  margin: 5px 45px 5px 5px
}

.form-field {
  height: 50px;
  width: 250px;
  line-height: 30px;
  vertical-align: middle;
  padding: 10px;
  background-color: #fff;
  border: 1px solid #eee;
  border-top: 1px solid #fff;
  border-bottom: 1px solid #ddd;
  cursor: move
}

.form-line {
  padding: 20px 30px;
  border: 1px solid transparent;
  border-radius: 6px;
  transition: all .3s ease;
  transition-property: border-color, background-color;
  cursor: move
}

.form-line, .form-line.selected {
  background-color: #f8f9fa
}

button {
  padding: .8em 1.2em;
  border: 0;
  outline: 0;
  margin-bottom: .5em;
  vertical-align: middle;
  color: #eee;
  background-color: #6495ed;
  cursor: pointer
}

button[disabled] {
  opacity: .5
}

.form-submit-button {
  width: 100%;
  height: 40px
}

button, label {
  user-select: none
}

.label {
  margin-bottom: 5px
}

.field {
  cursor: auto
}

.field input, .field select, .field textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 3px;
  outline: none;
  transition: border-color .3s ease
}

.field input[type=checkbox], .field input[type=radio] {
  width: auto
}

.field input:focus, textarea:focus {
  border: 1px solid #80bdff;
  box-shadow: 0 0 3px 0 #80bdff
}

.field-group input, .field-group select, .field-group textarea {
  width: 49%;
  margin-right: 2%
}

input:last-child {
  margin-right: 0
}

.smooth-dnd-container {
  position: relative;
  min-height: 30px;
  min-width: 30px;
}

.smooth-dnd-container.horizontal {
  display: table;
}

.smooth-dnd-container.horizontal > .smooth-dnd-stretcher-element {
  display: inline-block;
}

.smooth-dnd-container.horizontal > .smooth-dnd-draggable-wrapper {
  height: 100%;
  display: table-cell;
  vertical-align: top;
}

.smooth-dnd-container.vertical > .smooth-dnd-draggable-wrapper {
  overflow: hidden;
  display: block;
}

.smooth-dnd-draggable-wrapper {
  box-sizing: border-box;
}

.smooth-dnd-draggable-wrapper.horizontal {
  height: 100%;
  display: table-cell;
  vertical-align: top;
}

.smooth-dnd-draggable-wrapper.vertical {
  overflow: hidden;
  display: block;
}

.smooth-dnd-draggable-wrapper.animated {
  transition: transform ease;
}

.smooth-dnd-ghost {
  box-sizing: border-box;
}

.smooth-dnd-ghost.animated {
  transition: all ease-in-out;
}

.smooth-dnd-ghost * {
  pointer-events: none;
}

.smooth-dnd-disable-touch-action * {
  touch-action: none;
  -ms-touch-action: none;
}

.smooth-dnd-no-user-select {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.smooth-dnd-drop-preview-inner-class {
  flex: 1;
}

.smooth-dnd-container.horizontal > .smooth-dnd-drop-preview-constant-class {
  height: 100%;
  overflow: hidden;
  display: table-cell;
  vertical-align: top;
}

.smooth-dnd-container.vertical > .smooth-dnd-drop-preview-constant-class {
  overflow: hidden;
  display: block;
  width: 100%;
}

.smooth-dnd-drop-preview-flex-container-class {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: stretch;
  align-items: stretch;
}

.smooth-dnd-drop-preview-default-class {
  background-color: rgba(150, 150, 150, 0.1);
  border: 1px solid #ccc;
}
</style>
