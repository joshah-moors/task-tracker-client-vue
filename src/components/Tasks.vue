<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-10">
        <h1>Tasks</h1>
        <hr><br><br>
        <alert :message="message" v-if="showMessage"></alert>
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
                  <button type="button"
                          class="btn btn-warning btn-sm"
                          v-b-modal.task-update-modal
                          @click="editTask(task)">
                    Update
                  </button>
                  <button type="button"
                          class="btn btn-danger btn-sm"
                          @click="onDeleteTask(task)">
                    Delete
                  </button>
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
    <b-modal ref="editTaskModal"
             id="task-update-modal"
             title="Update"
             hide-footer>
      <b-form @submit="onSubmitUpdate" @reset="onResetUpdate" class="w-100">
        <b-form-group id="form-title-edit-group"
                    label="Title:"
                    label-for="form-title-edit-input">
          <b-form-input id="form-title-edit-input"
                        type="text"
                        v-model="editForm.title"
                        required
                        placeholder="Enter title">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-owner-edit-group"
                      label="Owner:"
                      label-for="form-owner-edit-input">
          <b-form-input id="form-owner-edit-input"
                        type="text"
                        v-model="editForm.owner"
                        required
                        placeholder="Enter owner">
          </b-form-input>
        </b-form-group>
        <b-form-group id="form-complete-edit-group">
          <b-form-checkbox-group v-model="editForm.complete" id="form-checks">
            <b-form-checkbox value="true">Complete?</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
        <b-button-group>
          <b-button type="submit" variant="primary">Update</b-button>
          <b-button type="reset" variant="danger">Cancel</b-button>
        </b-button-group>
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
      message: '',
      showMessage: false,
      editForm: {
        id: '',
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
          this.message = 'Task added!';
          this.showMessage = true;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.log(error);
          this.getTasks();
        });
    },
    editTask(task) {
      this.editForm = task;
    },
    onSubmitUpdate(evt) {
      evt.preventDefault();
      this.$refs.editTaskModal.hide();
      let complete = false;
      if (this.editForm.complete[0]) complete = true;
      const payload = {
        title: this.editForm.title,
        owner: this.editForm.owner,
        complete,
      };
      this.updateTask(payload, this.editForm.id);
    },
    updateTask(payload, taskID) {
      const path = `http://localhost:5000/tasks/${taskID}`;
      axios.put(path, payload)
        .then(() => {
          this.getTasks();
          this.message = 'Task updated!';
          this.showMessage = true;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
          this.getTasks();
        });
    },
    onDeleteTask(task) {
      this.removeTask(task.id);
    },
    removeTask(taskID) {
      const path = `http://localhost:5000/tasks/${taskID}`;
      axios.delete(path)
        .then(() => {
          this.getTasks();
          this.message = 'Task removed!';
          this.showMessage = true;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
          this.getTasks();
        });
    },
    initForm() {
      this.addTaskForm.title = '';
      this.addTaskForm.owner = '';
      this.addTaskForm.complete = [];
      this.editForm.id = '';
      this.editForm.title = '';
      this.editForm.owner = '';
      this.editForm.complete = [];
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
    onResetUpdate(evt) {
      evt.preventDefault();
      this.$refs.editTaskModal.hide();
      this.initForm();
      this.getTasks();
    },
  },
  created() {
    this.getTasks();
  },
};
</script>
