<template>
  <div class="dashboard-container">
    <zyAudio
      :url="url"
    />
  </div>
</template>

<script>
import zyAudio from './components/index'
export default {
  name: 'AudioIndex',
  components: {
    zyAudio
  },
  data() {
    return {
      // url: 'http://ia902606.us.archive.org/35/items/shortpoetry_047_librivox/song_cjrg_teasdale_64kb.mp3'
      // url: 'https://mindflowai-open.oss-cn-hangzhou.aliyuncs.com/505/6mariyokwzjuqjql.wav'
      url: require('./assets/music.mp3')
      // url: require('@/components/ZyAudio/assets/music.mp3')
    }
  },
  computed: {},
  watch: {
  },
  created() {
  },
  mounted() {

  },
  methods: {

  }
}
</script>

<style lang="scss" scoped>

.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
