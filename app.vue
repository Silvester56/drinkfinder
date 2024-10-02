<template>
  <div>
    <Header />
    <Question phrase="Salé ou sucré ?" :button-list="[{text: 'salé'}, {text: 'sucré'}]"/>
    <Footer />
  </div>
</template>

<script>

import { Footer } from "./components/Footer.vue";
import { Question } from "./components/Question.vue";
import { Header } from "./components/Header.vue";

</script>

<style>

body {
  margin: 0;
  padding: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

</style>