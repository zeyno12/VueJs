<template>
  <div class="todo-list">
    <h1>TodoList</h1>
    <input class="text-field" type="text" @change="addToList" v-model="text" />
    <div class="buttonsBox">
      <button
        v-on:click="makeRed"
        v-bind:class="{ red: isActive }"
        @click="filterAll"
      >
        All
      </button>
      <button
        v-on:click="makeRed"
        v-bind:class="{ red: isActive }"
        @click="filterSelected"
      >
        Selected
      </button>
      <button
        v-on:click="makeRed"
        v-bind:class="{ red: isActive }"
        @click="filterNoSelected"
      >
        No-Selected
      </button>
    </div>
    <ul>
      <li
        v-for="(item, index) in filteredList"
        :key="index"
        :class="{ new: item.isNew, old: !item.isNew }"
      >
        <span @click="toggleCheck(item)">
          <input type="checkbox" :checked="item.done" />
          <span :class="{ done: item.done }">{{ item.label }}</span>
        </span>
        <div v-if="editMode && editedIndex === index">
          <input type="text" v-model="list[index].label" />
          <button @click="saveEdit(index)">Save</button>
          <button @click="cancelEdit(index)">Cancel</button>
        </div>
        <div v-else>
          <span>Created: {{ item.created }}</span>
          <button @click="editItem(index)">Edit</button>
          <button @click="deleteFromList(index)">Delete</button>
        </div>
      </li>
    </ul>
    <div>
      <button @click="deleteSelected">Selected deleted</button>
      <button @click="deleteAll">All deleted</button>
    </div>
    <div class="svgBox" @click="toggleShowNewItems">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="16"
        height="16"
        fill="currentColor"
        class="bi bi-sort-down"
        viewBox="0 0 16 16"
      >
        <path
          d="M3.5 2.5a.5.5 0 0 0-1 0v8.793l-1.146-1.147a.5.5 0 0 0-.708.708l2 1.999.007.007a.497.497 0 0 0 .7-.006l2-2a.5.5 0 0 0-.707-.708L3.5 11.293V2.5zm3.5 1a.5.5 0 0 1 .5-.5h7a.5.5 0 0 1 0 1h-7a.5.5 0 0 1-.5-.5zM7.5 6a.5.5 0 0 0 0 1h5a.5.5 0 0 0 0-1h-5zm0 3a.5.5 0 0 0 0 1h3a.5.5 0 0 0 0-1h-3zm0 3a.5.5 0 0 0 0 1h1a.5.5 0 0 0 0-1h-1z"
        />
      </svg>
    </div>
    <div class="popup_box">
      <div v-if="showNewItems" class="popup">
        <div>
          <h2>Newest Items:</h2>
          <ul>
            <li v-for="(item, index) in newestItems" :key="index">
              <span>{{ item.label }}</span>
              <span>Created: {{ item.created }}</span>
            </li>
          </ul>
        </div>
      </div>
      <div v-if="showNewItems" class="popup">
        <div>
          <h2>Oldest Items:</h2>
          <ul>
            <li v-for="(item, index) in oldestItems" :key="index">
              <span>{{ item.label }}</span>
              <span>Created: {{ item.created }}</span>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "TodoList",
  data() {
    return {
      list: [],
      filterStatus: "all",
      text: "",
      editMode: false,
      editedIndex: -1,
      tempEditList: [],
      showNewItems: true,
      filteredList: [],
      newestItems: [],
      oldestItems: []
    };
  },
  created() {
    this.list = JSON.parse(localStorage.getItem("list")) || [];
    this.filterList();
    this.filterNewestOldestItems();
  },
  mounted() {
    this.filterList();
  },
  methods: {
    addToList() {
      const currentDate = new Date();
      const formattedDate = currentDate.toLocaleString();
      const newItem = {
        label: this.text,
        done: false,
        created: formattedDate,
        isNew: true
      };
      this.list.unshift(newItem);
      this.updateLocalStorage();
      this.text = "";
      this.filterNewestOldestItems();
    },
    toggleShowNewItems() {
      this.showNewItems = !this.showNewItems;
    },
    deleteFromList(index) {
      this.list.splice(index, 1);
      this.updateLocalStorage();
      this.filterNewestOldestItems();
    },

    toggleCheck(item) {
      item.done = !item.done;
      this.updateLocalStorage();
    },
    editItem(index) {
      this.editMode = true;
      this.editedIndex = index;
    },
    saveEdit(index) {
      this.list[index].created = new Date().toLocaleString(); // Update the "created" property.
      this.editMode = false;
      this.editedIndex = -1;
      this.updateLocalStorage();
    },
    cancelEdit(index) {
      this.editMode = false;
      this.editedIndex = -1;
      this.list = JSON.parse(JSON.stringify(this.tempEditList));
    },
    toggleShowNewItems() {
      this.showNewItems = !this.showNewItems;
      this.filterList();
    },
    filterList() {
      if (this.showNewItems) {
        this.filteredList = this.list.filter(item => item.isNew);
      } else {
        this.filteredList = this.list;
      }
    },
    makeRed: function() {
      this.isActive = !this.isActive;
    },
    filterAll() {
      this.filterStatus = "all";
      this.updateFilter(this.filterStatus);
    },
    filterSelected() {
      this.filterStatus = "selected";
      this.updateFilter(this.filterStatus);
    },
    filterNoSelected() {
      this.filterStatus = "noselected";
      this.updateFilter(this.filterStatus);
    },
    updateFilter(status) {
      if (status == "selected") {
        this.filteredList = this.list.filter(item => item.done);
      } else if (status == "noselected") {
        this.filteredList = this.list.filter(item => !item.done);
      } else {
        this.filteredList = this.list;
      }
    },
    filterNewestOldestItems() {
      this.newestItems = [...this.list].sort(
        (a, b) => new Date(b.created) - new Date(a.created)
      );
      this.oldestItems = [...this.list].sort(
        (a, b) => new Date(a.created) - new Date(b.created)
      );
    },
    deleteSelected() {
      this.list = this.list.filter(item => !item.done);
      this.updateLocalStorage();
    },

    deleteAll() {
      this.list = [];
      this.updateLocalStorage();
    },
    updateLocalStorage() {
      localStorage.setItem("list", JSON.stringify(this.list));
      this.updateFilter(this.filterStatus);
    }
  }
};
</script>

<style scoped>
.todo-list {
  border: 1px dashed red;
  padding: 30px;
  max-width: 90%;
  margin: 0 auto;
}
.red {
  border: 1px solid red;
}
h1 {
  text-align: center;
}

.buttonsBox {
  margin: 12px 0 0 0;
}
.text-field {
  width: 100%;
  height: 35px;
}
.popup {
  background-color: #9791ff;
  border: 1px solid #363434;
  padding: 10px;
  cursor: pointer;
  width: 50%;
}
.popup_box {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 20px;
}
.svgBox {
  width: 100%;
}
.bi-sort-down {
  cursor: pointer;
}
ul {
  list-style: none;
  padding: 0;
}

li {
  display: flex;
  padding: 10px 0;
  justify-content: space-between;
}

.done {
  text-decoration: line-through;
}
.new {
  background-color: #ffff99;
  padding: 5px;
}
.old {
  background-color: #ff9999;
  padding: 5px;
}
</style>
