<template>
  <div class="preview-card">
    
    <div class="date">
      <div style="display: flex;">
        <span style=" flex: 1 4 50%; text-align: left; padding-left: 10px;">{{formatDate(post.createdAt)}}</span>
        <div class="tags">
          <div class="tag" v-for="tag in tags" :key=tag>
          <span>{{ tag }}</span>
      </div>
    </div>
      </div>
    </div>
    <NuxtLink :to="{ name: 'posts-slug', params: { slug: post.slug } }">
      <h2 style = "flex: 0 1 25%; justify-content: center;">{{ post.title }}</h2>
    </NuxtLink>
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
  },
  computed: {
    tags: function () {
      return this.post.tags?.split(";") ?? [];
    }
  }
}
</script>

<style>
a {
  text-decoration: none;
}

a:visited {
  color: black;
}

a:hover {
  color: blue;
}

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
  flex: 0 0 35%;
  display: block;
  margin-left: auto;
  margin-right: auto;
}

h2 {
  flex: 0 0 25%;
}

.description {
  flex: 0 0 25%;
}

.date {
  flex: 0 0 5%;
}

.tags {
  flex: 1 4 50%;
  padding: 3px;
  margin: 3px;
}

.tag {
  padding: 3px;
  margin: 3px;
  display: inline;
  background-color: rgb(223 219 219);
  border-radius: 5px;
  font-size: 15px;
}

</style>