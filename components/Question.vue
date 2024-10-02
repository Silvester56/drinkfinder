<template>
    <div class="question">
        <h3 class="question-title">{{ phrase }}</h3>
        <button class="question-button" v-for="b in buttonList">{{ b.text }}</button>
    </div>
</template>

<script setup>
const props = defineProps({
  phrase: {
    type: String,
    required: true,
  },
  buttonList: {
    type: Array,
    required: true,
  }
});
</script>

<style>

.question {
    width: 100%;
    height: 50vh;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-evenly;
    align-items: center;
}

.question .question-title {
    width: 100%;
    text-align: center;
}

.question .question-button {
    background-color: #00dc82;
    border: none;
    color: white;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    cursor: pointer;
}

</style>