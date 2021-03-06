<template>
  <div class="bg-light py-2">
    <form id="tracker" class="container-fluid" @submit.prevent="submitEntry">
      <div class="row">
        <div class="col-sm-3 px-2" @keyup.enter.prevent>
          <selector
            id="tracker-client"
            v-model="client"
            :options="clients"
            :selected="client"
            placeholder="Student"
          />
        </div>

        <div class="col-sm-3 px-2" @keyup.enter.prevent>
          <selector
            id="tracker-project"
            v-model="project"
            :options="projects"
            :selected="project"
            placeholder="Class"
            required
          />
        </div>

        <div class="col-sm-3 px-2" @keyup.enter.prevent>
          <selector
            id="tracker-task"
            v-model="task"
            :options="tasks"
            :selected="task"
            placeholder="Teacher"
            required
          />
        </div>

        <div class="col-sm-3 px-2">
          <datepicker
            v-model="date"
            id="tracker-date"
            type="text"
            class="form-control date-input text-center"
            placeholder="Date"
          />
        </div>
      </div>

      <div class="row mt-2">
        <div class="col-sm-6 px-2">
          <input
            id="tracker-note"
            v-model="note"
            class="form-control w-100 shadow-sm"
            placeholder="Note"
            type="text"
          />
        </div>

        <div class="col-sm-3 px-2">
          <input
            id="tracker-duration"
            v-model="duration"
            class="form-control text-right font-weight-bold w-100 shadow-sm"
            placeholder="0:00"
            type="text"
            required
          />
        </div>

        <div class="col-sm-3 px-2">
          <button
            v-if="!running && !duration"
            class="btn btn-success w-100"
            @click.prevent.exact="toggle"
          >
            <icon :icon="['fas', 'play']" class="mr-1" />
            Start
          </button>
          <button
            v-else-if="running"
            class="btn btn-danger w-100"
            @click.prevent.exact="toggle"
          >
            <icon :icon="['fas', 'stop']" class="mr-1" />
            Stop ({{ seconds }})
          </button>
          <button
            v-else-if="!running && duration"
            :disabled="!canSubmit"
            id="tracker-submit"
            class="btn btn-info w-100"
            type="submit"
          >
            <icon :icon="['fas', 'plus']" class="mr-1" />
            Add
          </button>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";

import Selector from "./selector.vue";
import Datepicker from "./datepicker.vue";

function getFromLocalStorage(key, defaultValue) {
  if (!key in window.localStorage) {
    return defaultValue;
  }
  if (defaultValue === null && window.localStorage[key] === "null") {
    return null;
  }
  if (defaultValue === false) {
    return window.localStorage[key] === "true";
  }
  return window.localStorage[key];
}

export default {
  components: {
    Selector,
    Datepicker
  },
  data() {
    return {
      task: getFromLocalStorage("tracker_task", null),
      client: null,
      project: getFromLocalStorage("tracker_project", null),
      note: getFromLocalStorage("tracker_note", null),
      duration: null,
      datetimeStart:
        "tracker_start" in window.localStorage
          ? new Date(Number(window.localStorage.tracker_start))
          : null,
      datetimeEnd: null,
      date: this.$moment().format("YYYY-MM-DD"),

      running: getFromLocalStorage("tracker_running", false),
      total: 0,
      seconds: "0"
    };
  },
  computed: {
    ...mapGetters({
      clients: "clients/getSelectClients",
      tasks: "tasks/getSelectTasks",
      projectsAll: "projects/getSelectProjects",
      projectsClient: "projects/getSelectProjectsForClient",
      users: "users/getSelectUsers",
      getProject: "projects/getProject"
    }),
    projects() {
      if (!this.client) {
        return this.projectsAll;
      } else {
        return this.projectsClient(this.client);
      }
    },
    canSubmit() {
      return this.project && this.task;
    }
  },
  watch: {
    client() {
      if (this.project) {
        const p = this.getProject();
        if (p.client !== this.client) {
          this.project = null;
        }
      }
    },
    project() {
      window.localStorage.tracker_project = this.project;
    },
    task() {
      window.localStorage.tracker_task = this.task;
    },
    note() {
      window.localStorage.tracker_note = this.note;
    }
  },
  created() {
    if (this.running) {
      this.tick();
      this.start();
    }
  },
  methods: {
    ...mapActions({
      createEntry: "entries/createEntry"
    }),
    submitEntry() {
      this.createEntry({
        task: this.task,
        project: this.project,
        note: this.note,
        duration: this.duration,
        datetime_start: this.datetimeStart,
        datetime_end: this.datetimeEnd,
        user: timestrapConfig.USER.URL,
        date: this.date //this.$moment().format('YYYY-MM-DD'),
      });
      this.reset();
      this.note = null;
    },
    reset() {
      this.duration = null;
      this.total = 0;
      this.seconds = "0";
    },
    tick() {
      this.total = Math.floor(
        (Date.now() - this.datetimeStart.valueOf()) / 1000
      );
      this.seconds = (this.total % 3600) % 60;
      this.duration = this.$moment
        .duration(this.total, "seconds")
        .format("h:mm", { trim: false });
      document.title = this.duration + " — " + timestrapConfig.SITE.NAME;
    },
    start() {
      this.interval = setInterval(this.tick, 1000, this);
    },
    toggle() {
      this.running = !this.running;
      window.localStorage.tracker_running = this.running;

      if (this.running) {
        this.datetimeStart = new Date(Date.now());
        window.localStorage.tracker_start = this.datetimeStart.getTime();
        this.reset();
        this.start();
      } else {
        this.datetimeEnd = new Date(Date.now());
        window.localStorage.tracker_start = null;
        clearInterval(this.interval);
        if (this.total < 60) this.reset();
        document.title = "Application — " + timestrapConfig.SITE.NAME;
      }
    }
  }
};
</script>
