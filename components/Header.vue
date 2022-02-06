<template>
  <header class="header graded-backround">
      <nav style="position: sticky; display: flex; flex-direction: row-reverse; color: white; letter-spacing: 0.2rem;" class="graded-backround"> 
        <NuxtLink style="color: white; padding: 0.4rem; font-size: 1.1rem;" to="/about-me">ABOUT ME</NuxtLink>
        <NuxtLink style="color: white; padding: 0.4rem; font-size: 1.1rem;" to="/">POSTS</NuxtLink>
      </nav>
      <div class="title">
        <h1 style="font-size: 2rem;">jorgeolive.dev</h1>
        <div class="hide-mobile" style="font-size: 1.5rem;">A blog about software engineering by <b>Jorge Olivé Rodríguez</b></div>
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
  height: 15rem;
  font-family: Cabin, Arial, sans-serif;
}

.title {
  color: white;
  letter-spacing: 4px;
  text-align: center;
  padding-top: 25px;
  font-size: 2rem;
  top: 3rem;
}

@media screen and (max-width: 650px) {
  .hide-mobile {
    visibility: hidden;
  }
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