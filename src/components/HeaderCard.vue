<template>
    <div class="container">
        <h2>{{ categoryTitle }}</h2>
    </div>
</template>

<script>
export default {
    name: 'CategoryTitle',
    props: {
        categoryTitle: {
            type: String,
            required: true
        }
    }
}
</script>

<style scoped>
.container {
    display: flex;
    width: 100%;
    height: 75px;
    background-color: rgb(50, 50, 50);
    color: white;
    justify-content: center;
    align-items: center;
    border: 2px solid black;
}

h2 {
    text-align: center;
}
</style>