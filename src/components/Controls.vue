<template>
  <div class="controls action-buttons">
    <div
      class="button remove-button"
      :class="{'disabled': !selectedDate}"
      @click="$emit('handleRemoveBtn')"
    >
      Remove
    </div>
    <div
      class="button today-button"
      :class="{'disabled': !canSetToday}"
      @click="$emit('handleTodayBtn')"
    >
      Today
    </div>
  </div>
</template>

<script>
export default {
  name: "Controls",
  props: {
    canSetToday: {
      type: Boolean,
      default: true,
    },
    selectedDate: Date,
  },
}
</script>
