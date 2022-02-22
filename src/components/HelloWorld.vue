<!--
 * @Author: your name
 * @Date: 2022-02-22 16:23:14
 * @LastEditTime: 2022-02-22 16:42:17
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: \audio-play\src\components\HelloWorld.vue
-->
<template>
  <div class="hello">
    <AudioPlayer v-if="url" ref="record-audio" :src="url" layout="play, times, progress, hoverSound, download" />
    <el-slider v-model="value1" @change="changeA" @input="changeB" />
  </div>
</template>

<script>
import AudioPlayer from './AudioPlayer.vue'
export default {
  name: 'HelloWorld',
  components: {
    AudioPlayer
  },
  props: {
    msg: String
  },
  data() {
    return {
      url: '',
      value1: 0
    }
  },
  created() {
    this.url = require('../../public/ccc.mp3')
  },
  methods: {
    changeA(val) {
      console.log('[ val1]', val)
    },
    changeB(val) {
      console.log('[ val2 ]', val)
    }
  }
}
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
