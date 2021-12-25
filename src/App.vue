<template>
  <div>
    <div>
      <b-navbar toggleable="lg" type="dark" variant="info">
        <div class="navbar" href="#">
          Магазин товаров
        </div>
      </b-navbar>
    </div>
    <router-view></router-view>
  </div>
</template>

<style>
  body {
    margin: 0;
    padding: 0;
  }

  .navbar {
    margin: 0 auto;
    display: block;
    text-align: center;
    font-size: 24px;
    color: aliceblue;
  }
</style>
