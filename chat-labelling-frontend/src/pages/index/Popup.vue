<template>
  <div class="popup" v-if="showPopup">
    <div class="popup-content">
      <!-- 这里添加弹窗的内容 -->
      <p>又有新用户等待您的聊天，快去帮助吧(PS:您的角色定位不变，只打开另一个网页与新用户聊天)</p>
      <button @click="closePopup2">关闭</button>
      <button @click="closePopup">接受</button>
    </div>
  </div>
</template>

<script>
export default {
  props: ['touser', 'touuid'],
  data () {
    return {
      showPopup: false
    }
  },
  methods: {
    openPopup () {
      this.showPopup = true
    },
    closePopup () {
      this.showPopup = false
      window.open('/index?_uuid=' + this.touuid + '&_user=' + this.touser)
    },
    closePopup2 () {
      this.$emit('stop-timer')
      this.showPopup = false
    }
  }
}
</script>

<style scoped>
.popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.popup-content {
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}
</style>
