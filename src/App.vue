<template>
  <div class="app">
    <h1>Task Manager</h1>
    <div 
      v-for="task in tasks" 
      :key="task.id" 
      class="list" 
      draggable="true" 
      @dragstart="startDrag($event, task)"
    >
      <div 
        class="task-name"
        @drop="onDrop($event, task)" 
        @dragover.prevent 
        @dragenter.prevent>
        {{ task.name }}
        <button 
        class="delete-btn"
        @click="deleteTask(task.id)"
        >Delete
        </button>
        <label for="update-task"></label>
        <input 
          type="text" 
          :id="'update' + task.id"
          placeholder="Update the task name">
        <button 
          @click="updateTask(task.id)"
          class="update-btn"
        >
        Update
        </button>
      </div>
    </div>
    <form @submit.prevent="addTask">
      <label for="new-task">New Task:</label>
      <input type="text" id="new-task">
      <button>Add Task</button>
    </form>
    <div>
      <p>{{ this.updateText }}</p>
      <button 
        class="priority-btn" 
        @click="updateOrder"
      >Submit Priority Changes
      </button>
    </div>
  </div>
</template>



<script>
import axios from "axios";

export default {
  name: 'App',


  data() {
    return {
      tasks: [],
      newTask: "",
      updateText: "Click and drag to adjust priority of tasks (top = most important)",
      getTasksUrl: 'http://localhost:8000/api/v1/tasks',
      createTaskUrl: 'http://localhost:8000/api/v1/tasks',
      editTaskUrl: 'http://localhost:8000/api/v1/tasks/', // Append id here
      updateOrderUrl: 'http://localhost:8000/api/v1/tasks/order',
      deleteTaskUrl: 'http://localhost:8000/api/v1/tasks/', // Append id here

    };
  },

  methods: {

    startDrag(event, item) {
      event.dataTransfer.dropEffect = 'move';
      event.dataTransfer.effectAllowed = 'move';
      event.dataTransfer.setData('taskId', item.id);
      event.dataTransfer.setData('priority', item.priority);
      event.dataTransfer.setData('name', item.name);
    },


    onDrop(event, item) {
      const taskIdOne = event.dataTransfer.getData('taskId');
      const taskIdTwo = item.id;
      const priorityOne = event.dataTransfer.getData('priority');
      const priorityTwo = item.priority;
      const nameOne = event.dataTransfer.getData('name');
      const nameTwo = item.name;
      const task = this.tasks.find(task => task.id == taskIdOne);
      const taskTwo = this.tasks.find(task => task.id == taskIdTwo);
      task.priority = priorityTwo;
      task.name = nameTwo;
      taskTwo.priority = priorityOne;
      taskTwo.name = nameOne;
    },

    /**
     * Populate the page with tasks
     */
    async getTasks() {
      let config = {
        headers: {
          'Access-Control-Allow-Origin': '*'
        }
      }
      try {
        const { data } = await axios.get(this.getTasksUrl, config);
        this.tasks = data;
      } catch (e) {
        console.log(e);
      }
    },

    /**
    * Add a new task to the database
    */
    async addTask() {
      const input = document.getElementById("new-task").value;
      document.getElementById("new-task").value = "";
      try {
        const { data } = await axios.post(this.createTaskUrl, {
          "name": input,
          "priority": this.tasks.length + 1
        });

        this.tasks.push({
          "id": data.newId,
          "name": input,
          "priority": this.tasks.length + 1
        });
      } catch (e) {
        console.log(e);
      }
    },


    /**
     * Update a task in the database
     */
    async updateTask(id) {
      let input = document.getElementById("update" + id).value;
      document.getElementById("update" + id).value = "";
      try {
        const response = await axios.put(this.editTaskUrl + id, {
          "name": input
        },
        { headers: {
          Accept: "application/json"
        }});

        if (response.data.updated == 1) {
          this.getTasks();
        }

      } catch (e) {
        console.log(e);
      }
    },


    /**
     * Update the order of tasks
     */
    async updateOrder() {
      let idOrders = [];
      this.tasks = this.tasks.sort((a, b) => a.priority - b.priority);
      
      for (let i = 0; i < this.tasks.length; i++) {
        idOrders.push(this.tasks[i].id);
      }

      try {
        await axios.put(this.updateOrderUrl, {
          "idOrder": idOrders
        },
        { headers: {
          Accept: "application/json"
        }});
      } catch (e) {
        console.log(e);
      }

      this.getTasks();
      this.updateText = "Success!";
    },


    /**
     * Delete a task from the database
     */
    async deleteTask(id) {
      try {
        const response = await axios.delete(this.deleteTaskUrl + id);

        if (response.data.deleted == 1) {
          this.getTasks();
        }
      } catch (e) {
        console.log(e);
      }
    }
  },

  /**
   * On page load, get all the tasks from the database
   */
  beforeMount() {
    this.getTasks();
  },
}
</script>



<style>
.app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #2c3e50;
  text-align: center;
  margin-top: 60px;
  margin: auto;
}

.task-name {
  font-size: 2.0em;
}

.list {
  max-width: 60%;
  margin: 10px auto;
  background-color: #acbcc4;
  padding: 10px;
  min-height: 10px;
  color: white;
}

.zone {
  margin: 10px auto;
  padding: 10px 0px;
  width: 50%;
  background-color: rgb(206, 206, 206);
}

.priority-btn {
  margin-top: 20px;
}

button {
  border-radius: 5px;
  padding: 7px;
  color:#38385b;
  margin: 10px;
  font-weight: bold;
  background-color: #94b3f7;
}

.delete-btn {
  background-color: #0c1434;
  color: rgb(241, 241, 241);
}

.update-btn {
  background-color: #acbcc4;
  color: #38385b
}

input {
  padding: 8px;
  justify-items: right;
  border-radius: 6px;
}

html {
  background-color: rgb(235, 235, 235);
}
</style>
