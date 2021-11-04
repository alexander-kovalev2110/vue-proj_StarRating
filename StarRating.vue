<template>
  <div class="flex items-end" >
    <template v-for="index in 5">
      <svg
        :key="index"
        class="w-5 h-5 p-px fill-current cursor-pointer"
        :class="index <= ns ? 'text-red-500 hover:text-red-700' : 'text-gray-300 hover:text-gray-400'"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 32 32"
        v-on:click="() => {ns = index}"
      >
        <path d="M31.547 12a.848.848 0 00-.677-.577l-9.427-1.376-4.224-8.532a.847.847 0 00-1.516 0l-4.218 8.534-9.427 1.355a.847.847 0 00-.467 1.467l6.823 6.664-1.612 9.375a.847.847 0 001.23.893l8.428-4.434 8.432 4.432a.847.847 0 001.229-.894l-1.615-9.373 6.822-6.665a.845.845 0 00.214-.869z" />
      </svg>
    </template>
  </div>
</template>

<script lang="ts">
export default {
  name: "StarRating",
  props: {
    value: { type: Number, default: 0 }      // Original number of stars 0..5
  },
  data() {
    let ns = this.value > 5 ? 5 : this.value < 0 ? 0 : this.value     // Current number of stars 0..5
    return {ns}
  }
}
</script>
