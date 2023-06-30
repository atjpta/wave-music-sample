
<template>
    <div>
        <NuxtLink class="btn btn-outline btn-primary" to="/">chuyển sang home</NuxtLink>

        <div class="text-6xl">
            id:
            {{ id }}
        </div>
    </div>
</template>

<script setup>
const id = ref()
const route = useRoute()
id.value = route.params.id
</script>
