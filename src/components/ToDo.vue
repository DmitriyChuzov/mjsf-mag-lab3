<template>
  <div class="todo">
    <div class="card text-left">
      <div class="card-header">
        ToDo Component
      </div>
      <div class="card-body">
        <h5 class="card-title">My tasks</h5>
        <div class="card-text">
          <form>
            <div class="form-group">
              <label for="task-type">Type:</label>
              <select v-model="model.type" name="cars" class="form-control" id="task-type">
                <option value="critical">Critical</option>
                <option value="high">High</option>
                <option value="medium">Medium</option>
                <option value="low">Low</option>
              </select>

              <label for="task-title">Task title</label>
              <input v-model="model.title" type="text" class="form-control" id="task-title"
                     aria-describedby="emailHelp">
            </div>
            <div class="form-group">
              <label for="task-desc">Task description</label>
              <textarea v-model="model.description" class="form-control" id="task-desc"></textarea>
            </div>
            <button type="submit" class="btn btn-primary" @click.prevent="submit" :disabled="!isValid">Submit</button>
          </form>

          <ul class="pt-3">
            <li v-for="(item, index) in filteredTaskList" :key="index" class="d-flex justify-content-between mb-3">
              <div>
                <h4 :class="item.type"  v-if="item.status === 'completed'"><s>{{item.title}}</s></h4>
                <h4 :class="item.type"  v-else>{{item.title}}</h4>
                <p>{{item.description}}</p>
              </div>
              <div>
                <button class="btn btn-primary" @click="item.status = 'completed' ">Completed</button>
                <button class="btn btn-danger" @click.prevent="deleteItem(item)">Delete</button>
                <button class="btn btn-warning" @click="isOpen = true">Edit</button>
                <Modal :open="isOpen" @close="isOpen = !isOpen">
                <form>
                  <label for="task-type">Type:</label>
                  <select v-model="item.type" name="cars" class="form-control" id="task-type">
                    <option value="critical">Critical</option>
                    <option value="high">High</option>
                    <option value="medium">Medium</option>
                    <option value="low">Low</option>
                  </select>

                  <div class="form-group">
                    <label for="task-title">Task title</label>
                    <input v-model="item.title" type="text" class="form-control" id="task-title" 
                          aria-describedby="emailHelp">
                  </div>
                  <div class="form-group">
                    <label for="task-desc">Task description</label>
                    <textarea v-model="item.description" class="form-control" id="task-desc"></textarea>
                  </div>
                  <button type="submit" class="btn btn-primary" @click.prevent="editItem(item)" @click="isOpen = false">Submit</button>
                </form>
              </Modal>
              </div>

            </li>
          </ul>
        </div>
        <div class="card-footer">
          <button class="btn" :class="{'btn-primary':!filterStatus}" @click="filterStatus = ''">All</button>
          <button class="btn" :class="{'btn-primary': filterStatus === 'completed'}"
                  @click="filterStatus = 'completed'">Completed
          </button>
          <button class="btn" :class="{'btn-primary': filterStatus === 'incompleted'}"
                  @click="filterStatus = 'incompleted'">InCompleted
          </button>
          Active tasks count: {{ taskList.length }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import {Task} from '../models/task';
  import Modal from './Modal.vue';

  export default {
    name: "ToDo",
    props: {},
    data: () => {
      return {
        isOpen: false,
        model: new Task(),
        taskList: [],
        filterStatus: "",
        loading: false,
      }
    },
    async mounted() {
      this.loading=true;
      this.taskList= await this.$services.todo.fetch();
      this.loading = false;
    },
    methods: {
      submit() {
        this.$services.todo.save(this.model).then((res)=>{
          this.taskList.push(res);
          this.model = new Task();
        })
      },
      deleteItem(item) {
        this.$services.todo.delete(item).then((res)=>{
          this.taskList = res
        })
      },
      editItem(item) {
        this.$services.todo.edit(item).then((res)=>{
          this.taskList = res
        })
      }

    },
    watch: {
      message() {
        console.log("Nessage changed");
      }
    },
    computed: {
      messageLength() {
        return ("" + this.message).length;
      },
      isValid() {
        return this.model.title && this.model.description;
      },
      filteredTaskList() {
        return this.taskList.filter((item) => {
          if (!this.filterStatus) return true;
          return item.status === this.filterStatus;
        });
      }
    },
    components: {
      Modal
    }

  }
</script>

<style scoped>
  .todo {
    background-color: aqua;
  }

  .critical {
    background-color: red;
  }
  .high {
    background-color: pink;
  }

  .medium {
    background-color: green;
  }

  .low {
    background-color: blue;}
  
</style>
