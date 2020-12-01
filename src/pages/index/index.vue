<template>
  <view class="content">
    <calendar></calendar>
  </view>
</template>

<script>
import calendar from "@/components/calendar/index";
export default {
  data() {
    return {
      title: "Hello",
    };
  },
  onLoad() {},
  methods: {},
  components: {
    calendar,
  },
};
</script>

<style lang="scss" scoped>
.content {
  position: relative;
  width: 100%;
  height: 100%;
}
</style>
