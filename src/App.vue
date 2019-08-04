<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<style>
  html, body {
    height: 100%;
    margin: 0;
  }
  #app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #fff;
    background-color: #21D4FD;
    background-image: linear-gradient(15deg, #21D4FD 0%, #B721FF 100%);
    background-repeat: no-repeat;
    min-height: 100%;
  }
  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
    opacity: 0
  }
</style>
