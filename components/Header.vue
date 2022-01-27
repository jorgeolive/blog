<template>
  <header class="header graded-backround">
      <nav style="position: sticky; display: flex; flex-direction: row-reverse; color: white; letter-spacing: 2px;" class="graded-backround"> 
        <a style="padding: 10px;">CONTACT</a>
        <NuxtLink style="color: white; padding: 10px;" to="/about-me">ABOUT ME</NuxtLink>
        <NuxtLink style="color: white; padding: 10px;" to="/">POSTS</NuxtLink>
      </nav>
    <div style="color: white; letter-spacing: 4px; text-align: center; padding-top: 25px; font-size: 30px;">
      <h1 style=" top: 50px; font-size: 40px;">DISTRIBUTED CONCERNS</h1>
    </div>
    <div style="color: white; letter-spacing: 2px; text-align: center; font-size: 30px;">
      A blog about software engineering by <b>Jorge Olivé Rodríguez</b>
    </div>
  </header>
</template>

<script>
export default {
  name: "Header",
};
</script>

<style scoped>
.header {
  height: 250px;
  font-family: Cabin, Arial, sans-serif;
}

.graded-backround {
  top: 0;
  background: rgb(2 0 36);
  background:
    linear-gradient(
      90deg,
      rgb(2 0 36 / 100%) 14%,
      rgb(98 100 152 / 100%) 54%,
      rgb(36 38 38 / 100%) 81%
    );
}

a {
  text-decoration: none;
}

a:hover {
  font-weight: bold;
}

a.nuxt-link-exact-active {
  font-weight: bold;
}
</style>