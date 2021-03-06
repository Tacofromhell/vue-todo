<template>
  <section class="list">
    <header class="list__header" v-if="!isListNameEditable">
      <h2 class="list__name" @click="listNameEdit = !listNameEdit">
        {{ getName() }}
      </h2>
      <button
        class="btn btn--delete btn--delete-big list__btn-delete"
        type="button"
        @click="deleteList"
      >
        X
      </button>
    </header>
    <header
      class="list__header list__input-container list__input-container--list-name"
      v-else
    >
      <input
        class="input"
        type="text"
        maxlength="64"
        minlength="1"
        :placeholder="getName()"
        v-on:keyup.enter="changeListName"
        v-model="listNameModel"
        @blur="(e) => (e.target.value = removeWhitespace(e.target.value))"
      />
      <button
        class="btn btn--handle-input"
        type="button"
        @click="changeListName"
      >
        Change
      </button>
    </header>
    <hr />
    <ul v-for="item in items" :key="item._id">
      <TodoItemComponent :todoItem="item" @update-list="updateList" />
    </ul>
    <div class="list__input-container">
      <input
        class="input"
        type="text"
        maxlength="64"
        minlength="1"
        placeholder="New item?"
        v-model="newItemModel"
        v-on:keyup.enter="addItem"
        @blur="(e) => (e.target.value = removeWhitespace(e.target.value))"
      />
      <button class="btn btn--handle-input" type="button" @click="addItem">
        Add
      </button>
    </div>
  </section>
</template>

<script lang="ts">
import Vue, { PropType } from "vue";
import Component from "vue-class-component";
import TodoItemComponent from "@/components/TodoItem.vue";
import { TodoList } from "@/models/TodoList";
import { TodoItem } from "@/models/TodoItem";
import * as ItemService from "@/services/api/todoItemService";
import * as ListService from "@/services/api/todoListService";

const TodoListProps = Vue.extend({
  props: {
    todoList: Object as PropType<TodoList>,
  },
});

@Component({
  components: {
    TodoItemComponent,
  },
})
export default class TodoListComponent extends TodoListProps {
  newItemModel = "";
  list: TodoList = this.todoList;
  listNameEdit = false;
  listNameModel = this.todoList.name;

  get items(): TodoItem[] {
    return this.list.items;
  }
  get isListNameEditable() {
    return this.listNameEdit;
  }
  getName(): string {
    return this.list.name;
  }

  removeWhitespace(input: string): string {
    return input.replace(/\s\s+/g, " ").trim();
  }

  async addItem(): Promise<void> {
    if (this.newItemModel.length > 0 && this.todoList._id) {
      const newItem: TodoItem = await ItemService.addItem({
        list: this.todoList._id,
        name: this.removeWhitespace(this.newItemModel),
        checked: false,
      });
      if (newItem) {
        this.updateList();
        this.newItemModel = "";
      }
    }
  }

  async updateList(): Promise<void> {
    if (this.todoList._id) {
      const updatedList: TodoList = await ListService.getList(
        this.todoList._id,
      );
      if (updatedList) {
        this.list = updatedList;
      }
    }
  }
  async changeListName(): Promise<void> {
    if (this.listNameModel.length > 0 && this.todoList._id) {
      const updatedList: TodoList = await ListService.updateList(
        this.todoList._id,
        { ...this.list, name: this.removeWhitespace(this.listNameModel) },
      );
      if (updatedList.name) {
        this.list.name = updatedList.name;
        this.listNameEdit = false;
      }
    }
  }
  async deleteList(): Promise<void> {
    if (this.todoList._id) {
      const result: TodoList = await ListService.deleteList(this.todoList._id);
      if (result) {
        this.$emit("update-all");
      }
    }
  }
}
</script>

<style lang="less" scoped>
.list {
  background-color: lightgray;
  border: 0.25rem solid gray;
  padding: 1rem;
  border-radius: 10px;

  &__header {
    display: flex;
    justify-content: center;
    position: relative;
  }

  &__name {
    width: 100%;
    margin: 0.5rem;
    text-align: center;
    cursor: pointer;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
  }

  &__btn-delete {
    position: absolute;
    right: 0;
  }

  &__input-container {
    height: 2rem;
    display: flex;
    align-items: center;
    justify-content: center;
    padding-top: 1.5rem;

    &--list-name {
      padding: 1rem 0;
    }
  }
}
</style>
