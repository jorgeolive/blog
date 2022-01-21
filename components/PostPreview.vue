<template>
  <div class="preview-card">
      <h1>{{ post.title }}</h1>
      <nuxt-img class="img" v-if="post.image != null" v-bind:src="post.image" 
        width="250"
        height="250"/>
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
  min-height: 450px;
  padding: 0;
  background-color: rgb(255 255 255);
  font-family: MinionPro, Arial, sans-serif;
  justify-content: center;
  text-align: center;
  border-radius: 5px;
  box-shadow: 0 4px 8px 0 rgb(0 0 0 / 20%), 0 6px 20px 0 rgb(0 0 0 / 19%);
}

.img {
  display: block;
  margin-left: auto;
  margin-right: auto;
}

</style>