<template>
  <span class="text-red-800 text-right text-sm">
    {{ date | humanDate }}
  </span>
</template>

<script>
export default {
  props: {
    date: {
      type: String,
      default: null
    }
  },
  filters: {
    humanDate: function(value) {
      console.log(value);
      return Intl.DateTimeFormat("gb-EN", {
        day: "numeric",
        dayPeriod: "short",
        hour: "numeric",
        minute: "numeric",
        month: "long",
        second: "numeric",
        timeZoneName: "short",
        weekday: "long",
        year: "numeric"
      }).format(new Date(value));
    }
  }
};
</script>
