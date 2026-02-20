<template>
  <div id="app" class="mt-5">
    <section class="todoapp">
      <PageTitle title="To-Do List with Real-Time Updates" />
      <Todo />
    </section>
  </div>
</template>

<script>
import Todo from "./components/Todo.vue";
import PageTitle from "./components/PageTitle.vue";

export default {
  name: "App",
  components: {
    Todo,
    PageTitle
  }
};
</script>

