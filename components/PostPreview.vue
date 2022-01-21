<template>
  <div class="preview-card">
      <h1>{{ post.title }}</h1>
      <nuxt-img class="img" v-if="post.image != null" v-bind:src="post.image" 
        width="700"
        height="400"/>
      <span>{{ post.description }}</span>
  </div>
</template>

<script>
export default {
    name: "post-preview",
    props: {
    post : Object
    }
}
</script>

<style>
.preview-card {
  border-color: cadetblue;
  background-color: rgb(228 224 224);
  font-family: MinionPro, Arial, sans-serif;
  justify-content: center;
  text-align: center;
}

.img {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
</style>