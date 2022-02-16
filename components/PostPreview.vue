<template>
    <div class="preview-card">
      <div class="header">
        <span class="date">{{post.date}}</span>
        <div class="tags">
          <div class="tag" v-for="tag in tags" :key=tag>
            <span>{{ tag }}</span>
          </div>
        </div>
      </div>
      <NuxtLink :to="{ name: 'posts-slug', params: { slug: post.slug } }">
        <h2>{{ post.title }}</h2>
        <nuxt-img class="img" v-if="post.image != null" v-bind:src="post.image" sizes="sm:300px md:500px lg:600px xl:1000px 2xl: 1800px"/>
      </NuxtLink>
      <span class="description">{{ post.description }}</span>
    </div>
</template>

<script>
export default {
  name: "post-preview",
  props: {
  post : Object
  },
  computed: {
    tags: function () {
      return this.post.tags?.split(";") ?? [];
    }
  }
}
</script>

<style scoped>
a {
  text-decoration: none;
  font-size: 1.45rem;
  padding-left: 10px;
  padding-right: 10px;
}

a:visited {
  color: black;
}

.date {
  flex: 1 4 50%;
  text-align: left;
  padding-left: 10px;
  padding-top: 5px;
  color: gray;
}

.preview-card {
  display: flex;
  flex-direction: column;
  padding: 0;
  margin: 0.8rem;
  background-color: rgb(255 255 255);
  font-family: MinionPro, Arial, sans-serif;
  align-items: stretch;
  text-align: center;
  border-radius: 0.3rem;
  box-shadow: 0 4px 8px 0 rgb(0 0 0 / 20%), 0 6px 20px 0 rgb(0 0 0 / 19%);
}

.preview-card:hover {
  animation-name: hovered-post;
  animation-duration: 1s;
  animation-fill-mode: forwards;
}

@keyframes hovered-post {
  from {
    background-color: rgb(255 255 255);
  }

  to {
    background-color: rgb(230 230 230);
    padding: 0.1rem;
    transform: scale(1.02);
  }
}

.img {
  display: block;
  margin-left: auto;
  margin-right: auto;
}

h2 {
  margin: 0;
  justify-content: center;
  padding: 0.5rem;
}

.description {
  padding: 1.5rem;
  font-size: 1.2rem;
}

.header {
  display: flex;
  align-items: baseline;
  flex: 0 1 content;
  padding: 0.5rem;
  font-size: 1.2rem;
}

.tags {
  display: flex;
  flex: 1 4 35%;
  padding: 0.2rem;
  margin: 0.2rem;
  justify-content: flex-end;
}

.tag {
  padding: 3px;
  margin: 3px;
  display: inline;
  background-color: rgb(223 219 219);
  border-radius: 5px;
  font-size: 1.1rem;
}

</style>