<template>
    <div class="app">
        <input type="text" :value=user.name
        @input="user.name=$event.target.value">
    </div>
</template>
