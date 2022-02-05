<template>
  <div class="preview-card">
    <div class="header">
      <div style="display: flex; align-items: baseline;">
        <span style=" flex: 1 4 50%; text-align: left; padding-left: 10px; padding-top: 5px;">{{formatDate(post.createdAt)}}</span>
        <div class="tags">
          <div class="tag" v-for="tag in tags" :key=tag>
            <span>{{ tag }}</span>
          </div>
        </div>
      </div>
    </div>
    <NuxtLink :to="{ name: 'posts-slug', params: { slug: post.slug } }">
      <h2>{{ post.title }}</h2>
    </NuxtLink>
    <nuxt-img class="img" v-if="post.image != null" v-bind:src="post.image" height = "250" sizes="sm:320px md:650px lg:1024px xxl:1000px"/>
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
  font-size: 25px;
  padding-left: 10px;
  padding-right: 10px;
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
  min-height: 125px;
  margin: 0;
  justify-content: center;
  padding: 0;
}

.description {
  flex: 0 0 25%;
  padding: 3px;
  font-size: 20px;
  min-height: 50px;
}

.header {
  flex: 0 1 5%;
  padding: 0;
  align-items: center;
  font-size: 20px;
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
  font-size: 18px;
}

</style>