<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" /><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
      <div v-if="saveError" class="error-message">{{ saveErrorMessage }}</div>
    </form>

    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        {{ article.title }}<br />
        {{ article.content }}<br />
        <button @click="edit(article)">Edit</button>
        <button @click="deleteArticle(article.id)">Delete</button>
      </li>
    </ul>

    <button @click="load" class="load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: '',
      title: '',
      content: '',
    });

    const articles = ref([]);
    const saveError = ref(false); // Initially set to false
    const saveErrorMessage = ref('');

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
        console.log('Articles loaded:', articles.value);
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id ? `http://localhost:3000/articles/${form.id}` : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        console.log('Saving article:', { title: form.title, content: form.content });
        const response = await axios[method](url, { title: form.title, content: form.content });
        console.log('Save response:', response.data);

        // Update UI only if save is successful
        if (response.status === 200 || response.status === 201) {
          if (form.id) {
            articles.value = articles.value.map((article) =>
              article.id === response.data.id ? response.data : article
            );
          } else {
            articles.value.push(response.data);
          }

          // Reset form
          form.id = '';
          form.title = '';
          form.content = '';
          saveError.value = false;
          saveErrorMessage.value = '';
        } else {
          // Handle save error
          saveError.value = true;
          if (response.data && response.data.message) {
            saveErrorMessage.value = response.data.message;
          } else {
            saveErrorMessage.value = 'An error occurred while saving.';
          }
        }
      } catch (error) {
        console.error('Error saving article:', error);
        saveError.value = true;
        saveErrorMessage.value = 'An error occurred while saving.';
      }
    }

    async function deleteArticle(id) {
      try {
        // Remove the article from the local array first
        articles.value = articles.value.filter((article) => article.id !== id);
        // Then send delete request to the server
        await axios.delete(`http://localhost:3000/articles/${id}`);
        console.log('Article deleted:', id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load, saveError, saveErrorMessage };
  },
};
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.form {
  margin-bottom: 20px;
}

.form input[type="text"],
.form textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.form button {
  background-color: #007bff;
  color: #fff;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form button:hover {
  background-color: #0056b3;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #fff;
  margin-bottom: 10px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.05);
}

.article-item button {
  background-color: #dc3545;
  color: #fff;
  padding: 5px 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 5px;
}

.article-item button:hover {
  background-color: #c82333;
}

.article-item button:nth-child(2) {
  background-color: #28a745;
}

.article-item button:nth-child(2):hover {
  background-color: #218838;
}

.load-button {
  background-color: #17a2b8;
  color: #fff;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  display: block;
  width: 100%;
  text-align: center;
}

.load-button:hover {
  background-color: #138496;
}
</style>
