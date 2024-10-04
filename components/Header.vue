<template>
    <header>
        <h1>Drink Finder</h1>
    </header>
</template>

<script>

export default {
    name: "Header"
}

</script>

<style>

header {
    width: 100%;
    height: 100px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #e5e5f7;
    opacity: 0.8;
    background: repeating-linear-gradient( -45deg, #00dc82, #00dc82 5px, #e5e5f7 5px, #e5e5f7 25px );
}

</style>