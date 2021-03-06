<style lang="stylus">

@import "./base.css"
@import "./index.css"

[v-cloak]
  display: none

</style>

<template>

<div>
  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
    </header>
    <section class="main" v-show="todos.length" v-cloak>
      <input class="toggle-all" type="checkbox" v-model="allDone">
      <ul class="todo-list">
        <li class="todo" v-for="todo in filteredTodos" :class="{completed: todo.completed, editing: todo == editedTodo}">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{todo.title}}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input class="edit" type="text" v-model="todo.title" v-todo-focus="todo == editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length" v-cloak>
      <span class="todo-count"><strong v-text="remaining"></strong> {{remaining | pluralize 'item'}} left</span>
      <ul class="filters">
        <li><a href="#/all" :class="{selected: visibility == 'all'}">All</a></li>
        <li><a href="#/active" :class="{selected: visibility == 'active'}">Active</a></li>
        <li><a href="#/completed" :class="{selected: visibility == 'completed'}">Completed</a></li>
      </ul>
      <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
        Clear completed
      </button>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
    <p>Written by <a href="http://evanyou.me">Evan You</a></p>
    <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
  </footer>
</div>

</template>

<script>

import Vue from 'vue'
import store from '../store/StoreApi.js'

const filters = {
  all: function (todos) {
    return todos;
  },
  active: function (todos) {
    return todos.filter(function (todo) {
      return !todo.completed;
    });
  },
  completed: function (todos) {
    return todos.filter(function (todo) {
      return todo.completed;
    });
  }
};

export default {
  data() {
    return {
      // todos: store.fetch(),
      todos: [],
      newTodo: '',
      editedTodo: null,
      visibility: 'all'
    }
  },
  watch: {
    'todos': {
      deep: true, // 为了发现对象内部值的变化
      handler: store.save
    }
  },
  // 实例计算属性
  computed: {
    filteredTodos: function () {
      return filters[this.visibility](this.todos);
    },
    remaining: function () {
      return filters.active(this.todos).length;
    },
    allDone: {
      get: function () {
        return this.remaining === 0;
      },
      set: function (value) {
        this.todos.forEach(function (todo) {
          todo.completed = value;
        });
      }
    }
  },
  created () {
    // console.log('created');
  },
  route: {
    data({ to }) {
      store.fetch().then(todos => {
        this.todos = todos
      })
    }
  },
  methods: {
    addTodo: function() {
      let value = this.newTodo && this.newTodo.trim();
      if (!value) {
        return;
      }
      this.todos.push({
        title: value,
        completed: false
      });
      this.newTodo = '';
    },

    removeTodo: function(todo) {
      this.todos.$remove(todo);
    },

    editTodo: function(todo) {
      this.beforeEditCache = todo.title;
      this.editedTodo = todo;
    },

    doneEdit: function(todo) {
      if (!this.editedTodo) {
        return;
      }
      this.editedTodo = null;
      todo.title = todo.title.trim();
      if (!todo.title) {
        this.removeTodo(todo);
      }
    },

    cancelEdit: function(todo) {
      this.editedTodo = null;
      todo.title = this.beforeEditCache;
    },

    removeCompleted: function() {
      this.todos = filters.active(this.todos);
    }
  },
  directives: {
    'todo-focus': function (value) {
      if (!value) {
        return;
      }
      let el = this.el;
      Vue.nextTick(function () {
        el.focus();
      });
    }
  }
}

</script>
