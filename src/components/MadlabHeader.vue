<template>
  <header>
    <nav>
      <a href="http://madlabgroup.com/" target="_blank">
        <img src="../assets/logo.png" alt="Madlab Group" />
      </a>
    </nav>
    <h1>What can I realistically expect to earn in the Madlab system?</h1>
    <p>As an apprentice, you will learn under the direction of an experienced coach what it takes to be a professional. You will learn not only how to train your clients, but also how to build relationships with them and help them to reach their goals. As a professional, you have an amazing opportunity to earn a great living doing something that you love. We have the systems in place to do it, but it is up to you to set some realistic goals, and work hard to achieve them.</p>
  </header>
</template>

<script>
export default {
  name: 'MadlabHeader',
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
header {
  border-bottom:2px solid #777;
  nav {
    display: flex;
    justify-content: space-between;

    > a img {
      height: 16vw;
      max-height:90px;
    }
    
    > img {
      max-height: 24vw;
      height: 140px;
      margin-top: -20px;
    }
  }
  
}
</style>
