<template>
  <div>
    <input type="text" v-model="newList" class="todo-input" placeholder="Add a new list and press enter"
           @keyup.enter="addList">
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
    }
  }
}
</script>

<style scoped>

</style>
