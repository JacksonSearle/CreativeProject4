<template>
<div class="admin">
  <div class="px-4 py-5 my-5 text-center">
    <h1 class="display-5 fw-bold">Add More!</h1>
    <div class="col-lg-6 mx-auto">
      <p class="lead mb-4">Add an Animal</p>
      <input v-model="title" placeholder="How Fast is it? (in mph)"><br>
      <input type="file" name="photo" @change="fileChanged">
      <button @click="upload">Upload</button> <br>
      <div class="upload" v-if="addItem">
        <h2>{{addItem.title}}</h2>
        <img :src="addItem.path" />
      </div>
    </div>
  </div>
  <div class="px-4 py-5 my-5 text-center">
    <h1 class="display-5 fw-bold">Edit or Delete an Animal from the list</h1>
    <div class="col-lg-6 mx-auto">
      <div class="form">
        <input v-model="findTitle" placeholder="Search">
        <div v-if="suggestions.length > 0">
          <div v-for="s in suggestions" :key="s.id" @click="selectItem(s)">{{s.title}}
          </div>
        </div>
      </div>
      <div v-if="findItem">
        <input v-model="findItem.title">
        <img :src="findItem.path" />
      </div>
      <div v-if="findItem">
        <button @click="deleteItem(findItem)">Delete</button>
        <button @click="editItem(findItem)">Edit</button>
      </div>
    </div>
  </div>
</div>
</template>

<style scoped>
.image h2 {
  font-style: italic;
  font-size: 1em;
}

.heading {
  display: flex;
  margin-bottom: 20px;
  margin-top: 20px;
}

.heading h2 {
  margin-top: 8px;
  margin-left: 10px;
}

.add,
.edit {
  display: flex;
}

.circle {
  border-radius: 50%;
  width: 18px;
  height: 18px;
  padding: 8px;
  background: #333;
  color: #fff;
  text-align: center
}

/* Form */
input,
textarea,
select,
button {
  font-family: 'Montserrat', sans-serif;
  font-size: 1em;
}

.form {
  margin-right: 50px;
}

/* Uploaded images */
.upload h2 {
  margin: 0px;
}

.upload img {
  max-width: 300px;
}

/* Suggestions */
.suggestions {
  width: 200px;
  border: 1px solid #ccc;
}

.suggestion {
  min-height: 20px;
}

.suggestion:hover {
  background-color: #5BDEFF;
  color: #fff;
}
</style>

<script>
import axios from 'axios';
export default {
  name: 'Admin',
  data() {
    return {
      title: "",
      file: null,
      addItem: null,
      items: [],
      findTitle: "",
      findItem: null,
    }
  },
  created() {
    this.getItems();
  },
  computed: {
    suggestions() {
      let items = this.items.filter(item => item.title.toLowerCase().startsWith(this.findTitle.toLowerCase()));
      return items.sort((a, b) => a.title > b.title);
    }
  },
  methods: {
    selectItem(item) {
      this.findTitle = "";
      this.findItem = item;
    },
    async deleteItem(item) {
      try {
        await axios.delete("/api/items/" + item._id);
        this.findItem = null;
        this.getItems();
        return true;
      } catch (error) {
        console.log(error);
      }
    },
    async editItem(item) {
      try {
        await axios.put("/api/items/" + item._id, {
          title: this.findItem.title,
        });
        this.findItem = null;
        this.getItems();
        return true;
      } catch (error) {
        console.log(error);
      }
    },
    async getItems() {
      try {
        let response = await axios.get("/api/items");
        this.items = response.data;
        return true;
      } catch (error) {
        console.log(error);
      }
    },
    fileChanged(event) {
      this.file = event.target.files[0]
    },
    async upload() {
      try {
        const formData = new FormData();
        formData.append('photo', this.file, this.file.name)
        let r1 = await axios.post('/api/photos', formData);
        let r2 = await axios.post('/api/items', {
          title: this.title,
          path: r1.data.path
        });
        this.addItem = r2.data;
      } catch (error) {
        console.log(error);
      }
    },
  },
}
</script>