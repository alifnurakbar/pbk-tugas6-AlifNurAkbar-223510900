<template>
  <div class="container mt-5">
    <h1 class="mb-4">Article Manager</h1>
    <form @submit.prevent="save" class="mb-4">
      <div class="form-group">
        <label for="title">Title</label>
        <input type="text" id="title" v-model="form.title" class="form-control" />
      </div>
      <div class="form-group">
        <label for="content">Content</label>
        <textarea id="content" v-model="form.content" class="form-control"></textarea>
      </div>
      <div class="form-group">
        <label for="imageUrl">Image URL</label>
        <input type="text" id="imageUrl" v-model="form.imageUrl" class="form-control" />
      </div>
      <button type="submit" class="btn btn-primary">Save</button>
    </form>
    <ul class="list-group">
      <li v-for="article in articles" :key="article.id" class="list-group-item d-flex justify-content-between align-items-center">
        <div>
          <img :src="article.imageUrl" alt="Article Image" class="article-image" v-if="article.imageUrl"/>
          <h5>{{ article.title }}</h5>
          <p>{{ article.content }}</p>
        </div>
        <div>
          <button @click="edit(article)" class="btn btn-secondary btn-sm">Edit</button>
          <button @click="remove(article.id)" class="btn btn-danger btn-sm">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="btn btn-success mt-3">Load Articles</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
      imageUrl: ''
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = '';
        form.content = '';
        form.imageUrl = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
      form.imageUrl = article.imageUrl;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: auto;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  color: #ddd;
}

.list-group-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 0.1);
  border: none;
  border-radius: 5px;
  margin-bottom: 10px;
  padding: 10px 15px;
  transition: background 0.3s;
}

.list-group-item:hover {
  background: rgba(255, 255, 255, 0.2);
}

.btn {
  margin-right: 0.5rem;
}

.article-image {
  max-width: 100%;
  height: auto;
  border-radius: 5px;
  margin-bottom: 10px;
  transition: transform 0.3s;
}

.article-image:hover {
  transform: scale(1.05);
}
</style>
