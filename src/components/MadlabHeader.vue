<template>
  <header>
    <nav>
      <a href="http://madlabgroup.com/" target="_blank">
        <img src="../assets/logo.png" alt="MadLab Group" />
      </a>
      <img src="../assets/moosecock_rooster_final.png" alt="Moose Cock" />
    </nav>
    <h1>What can I realistically expect to earn in the MadLab system?</h1>
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
      height:90px;
    }
    
    > img {
      height:140px;
    }
  }
  
}
</style>
