<template>
  <div class="preview-card">
      <span class="date">{{formatDate(post.createdAt)}}</span>
      <h2 style = "flex: 0 1 25%; justify-content: center;">{{ post.title }}</h2>
      <nuxt-img class="img" v-if="post.image != null" v-bind:src="post.image" 
        width="400"
        height="250"/>
      <span class="description">{{ post.description }}</span>
  </div>
</template>

<script>
export default {
  name: "post-preview",
  props: {
  post : Object
  },
  methods: {
    formatDate(date) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' }
      return new Date(date).toLocaleDateString('en', options)
    }
  }
}
</script>

<style>
.preview-card {
  display: flex;
  flex-direction: column;
  min-height: 450px;
  padding: 0;
  margin: 10px;
  background-color: rgb(255 255 255);
  font-family: MinionPro, Arial, sans-serif;
  justify-content: center;
  text-align: center;
  border-radius: 5px;
  box-shadow: 0 4px 8px 0 rgb(0 0 0 / 20%), 0 6px 20px 0 rgb(0 0 0 / 19%);
}

.img {
  flex: 0 0 40%;
  display: block;
  margin-left: auto;
  margin-right: auto;
}

h2 {
  flex: 0 0 30%;
}

.description {
  flex: 0 0 25%;
}

.date {
  flex: 0 0 5%;
}

</style>