<template>
    <div class="cocktail">
        <h4 class="cocktail-name">{{ recipe.name }}</h4>
        <i>{{ recipe.category }}</i>
        <ul>
            <li v-for="i in recipe.ingredients">{{ i.direction }}</li>
        </ul>
        <p>{{ recipe.method }}</p>
        <p>{{ recipe.garnish }}</p>
    </div>
</template>

<script>

export default {
  props: ['recipe'],
  name: 'cocktail'
}
</script>

<style>

.cocktail {
    height: calc(100vh - 200px);
}

</style>