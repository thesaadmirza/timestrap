<template>
<cards
  :number-of-elements="tasks(search).length"
  :view_perm="$perms.view_task"
  @search="search = $event">
  <template slot="cards-title">
    <icon :icon="['fas', 'tasks']" class="fa-sm mr-2"/>
    Teachers
  </template>

  <template slot="cards-list">
    <task
      v-for="(task, index) in tasks(search)"
      :key="task.id"
      :task="task"
      :index="index"
      @modal="modalToggle(task)"/>
  </template>

  <template slot="cards-new">
    <task @modal="modalToggle()"/>
  </template>

  <task-modal
    slot="cards-modal"
    v-if="modalShow"
    :task="modalTask"
    @close="modalShow = false"/>
</cards>
</template>


<script>
import {mapGetters, mapActions} from 'vuex';

import Cards from '../cards/cards.vue';
import Task from './task.vue';
import TaskModal from './task-modal.vue';


export default {
  components: {
    Cards,
    Task,
    TaskModal,
  },
  data() {
    return {
      search: '',
      modalTask: null,
      modalShow: false,
    };
  },
  computed: {
    ...mapGetters({
      tasks: 'tasks/getSearchTasks',
    }),
  },
  methods: {
    modalToggle: function(task) {
      if (typeof(task) !== 'undefined') this.modalTask = task;
      else this.modalTask = null;
      this.modalShow = !this.modalShow;
    },
  },
};
</script>
