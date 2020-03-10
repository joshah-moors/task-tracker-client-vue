<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-10">
        <h1>Tasks</h1>
        <hr><br><br>
        <alert message="hi"></alert>
        <button type="button" class="btn btn-success btn-sm" v-b-modal.task-modal> Add Task</button>
        <br><br>
        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">Task</th>
              <th scope="col">Owner</th>
              <th scope="col">Complete?</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(task, index) in tasks" :key="index">
              <td>{{ task.title }}</td>
              <td>{{ task.owner }}</td>
              <td>
                <span v-if="task.complete">Yes</span>
                <span v-else>No</span>
              </td>
              <td>
                <div class="btn-group" role="group">
                  <button type="button" class="btn btn-warning btn-sm">Update</button>
                  <button type="button" class="btn btn-danger btn-sm">Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <b-modal ref="addTaskModal"
             id="task-modal"
             title="Add a new task"
             hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-title-group"
                      label="Title:"
                      label-for="form-title-input">
          <b-form-input id="form-title-input"
                        type="text"
                        v-model="addTaskForm.title"
                        required
                        placeholder="Enter title">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-owner-group"
                      label="Owner:"
                      label-for="form-owner-input">
          <b-form-input id="form-owner-input"
                        type="text"
                        v-model="addTaskForm.owner"
                        required
                        placeholder="Enter owner">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-read-group">
          <b-form-checkbox-group v-model="addTaskForm.read" id="form-checks">
            <b-form-checkbox value="true">Complete?</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
        <b-button type="submit" variant="primary">Submit</b-button>
        <b-button type="reset" variant="danger">Reset</b-button>
      </b-form>
    </b-modal>
  </div>
</template>


<script>
import axios from 'axios';
import Alert from './Alert.vue';

export default {
  data() {
    return {
      tasks: [],
      addTaskForm: {
        title: '',
        owner: '',
        complete: [],
      },
    };
  },
  components: {
    alert: Alert,
  },
  methods: {
    getTasks() {
      const path = 'http://localhost:5000/tasks';
      axios.get(path)
        .then((res) => {
          this.tasks = res.data.tasks;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
        });
    },
    addTask(payload) {
      const path = 'http://localhost:5000/tasks';
      axios.post(path, payload)
        .then(() => {
          this.getTasks();
        })
        .catch((error) => {
          // es-lint-disable-next-line
          console.log(error);
          this.getTasks();
        });
    },
    initForm() {
      this.addTaskForm.title = '';
      this.addTaskForm.owner = '';
      this.addTaskForm.complete = [];
    },
    onSubmit(evt) {
      evt.preventDefault();
      this.$refs.addTaskModal.hide();
      let complete = false;
      if (this.addTaskForm.complete[0]) complete = true;
      const payload = {
        title: this.addTaskForm.title,
        owner: this.addTaskForm.owner,
        complete, // property shorthand
      };
      this.addTask(payload);
      this.initForm();
    },
    onReset(evt) {
      evt.preventDefault();
      this.$refs.addTaskForm.hide();
      this.initForm();
    },
  },
  created() {
    this.getTasks();
  },
};
</script>
