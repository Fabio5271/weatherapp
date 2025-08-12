<template>
  <div class="extra-widget">
    <span class="title">{{ title }}</span><br>
    <span class="value">{{ val }}{{ unit }}</span>
  </div>
</template>

<script>
export default {
  name: 'ExtraWidget',
  props: {
    title: {
      type: String,
      required: true
    },
    val: {
      type: String,
      required: true
    },
    unit: {
      type: String,
      required: false
    }
  }
}
</script>

<style scoped>
.extra-widget {
  display: inline-block;
  padding: 5px 12px;
  margin: 15px 15px;
  text-align: center;
  font-size: 24px;
  font-weight: 500;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  color: #fff;
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 8px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.extra-widget .value {
  font-size: 50px;
  font-weight: 900;
}
</style>