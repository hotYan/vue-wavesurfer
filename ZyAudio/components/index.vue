<template>
  <div v-loading="loading" class="audio">
    <div class="audio-warp">
      <div class="waveform-warp">
        <div id="timeline" ref="timeline" />
        <div id="waveform" ref="waveform" />
      </div>

      <div v-if="!loading" class="button">
        <!-- 显示当前播放时间 -->
        <div v-show="showCurrentTime" class="button-warp" style="text-align:left;">
          <p>{{ '0 - 00:' +currentTime }}</p>
        </div>
        <!-- 操作按钮：开始，暂停等 -->
        <div class="button-warp">
          <el-button-group>
            <el-button
              v-for="item in playArr"
              :key="item.type"
              size="mini"
              :type="item.type"
              :icon="item.icon"
              @click="item.method"
            />
          </el-button-group>
        </div>
        <!-- 控制视图大小，音量大小 -->
        <div
          v-for="(item,index) in sliderArr"
          v-show="item.show"
          :key="index"
          class="button-warp"
          style="margin:20px 20px 0 20px;"
        >
          <img :src="item.left">
          <el-slider v-model="item.model" @change="item.method" />
          <img :src="item.right">
        </div>
        <!-- 切换速度 -->
        <div v-show="showSpeed" class="button-warp" style="float:right">
          <el-radio-group v-model="ratePosition" size="mini" @change="changeAudioRate">
            <el-radio-button
              v-for="item in rateArr"
              :key="item"
              :label="item"
            >{{ item }}
            </el-radio-button>
          </el-radio-group>
        </div>
      </div>

    </div>

  </div>
</template>

<script>
import '../assets/zy_icon/iconfont.css'
import WaveSurfer from 'wavesurfer.js'
import Timeline from 'wavesurfer.js/dist/plugin/wavesurfer.timeline'
import Region from 'wavesurfer.js/dist/plugin/wavesurfer.regions'
import Cursor from 'wavesurfer.js/dist/plugin/wavesurfer.cursor'
export default {
  name: 'Audio',
  components: {
  },
  props: {
    url: {
      type: String,
      default: ''
    },
    showRound: {
      type: Boolean,
      default: true
    },
    showCurrentTime: {
      type: Boolean,
      default: true
    },
    showSpeed: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      isReady: false,
      loading: true,
      currentTime: '00:00.00',
      sliderArr: [
        { left: require('@/components/ZyAudio/assets/images/zoomDown.png'), method: this.changeZoom, model: 50, right: require('@/components/ZyAudio/assets/images/zoomUp.png'), show: true },
        { left: require('@/components/ZyAudio/assets/images/volumeDown.png'), method: this.changeRound, model: 50, right: require('@/components/ZyAudio/assets/images/volumeUp.png'), show: this.showRound }
      ],
      playArr: [
        { type: 'success', icon: 'zy-icon-play', method: this.play },
        { type: 'warning', icon: 'zy-icon-pause', method: this.pause },
        { type: 'primary', icon: 'zy-icon-replay', method: this.rePlay },
        { type: 'danger', icon: 'zy-icon-stop', method: this.stop }
      ],
      ratePosition: '1.0',
      rateArr: ['0.5', '1.0', '1.25', '1.5', '2.0'],
      wavesurfer: ''
    }
  },

  computed: {},
  watch: {},
  created() {
    this.initAudio()
  },
  mounted() {},
  methods: {
    initLoading() {
      this.loading = false
    },
    initAudio(status = 0, url = '', data = []) {
      const _this = this
      if (_this.wavesurfer) {
        _this.wavesurfer.destroy()
        _this.wavesurfer = ''
      }
      _this.$nextTick(() => {
        _this.loading = true
        _this.wavesurfer = WaveSurfer.create({
          container: _this.$refs.waveform,
          // container: document.querySelector('#waveform'),
          // container: '#waveform',
          interact: false,
          audioRate: 1,
          audioContext: '加载中...',
          forceDecode: true,
          minPxPerSec: 10,
          barGap: 3,
          waveColor: '#A8DBA8',
          progressColor: '#A8DBA8',
          backend: 'MediaElement',
          plugins: [
            Region.create({
              regions: [],
              dragSelection: {
                // slop: 0
              }
            }),
            Timeline.create({
              container: _this.$refs.timeline,
              secondaryColor: '#FF0000', // 红色
              secondaryFontColor: '#FF0000',
              secondaryLabelInterval: _this._secondaryLabelInterval,

              primaryColor: '#3498DB', // 蓝色
              primaryFontColor: '#D3498DB',
              primaryLabelInterval: _this._primaryLabelInterval,
              formatTimeCallback: _this._formatTimeCallback,
              timeInterval: _this._timeInterval,
              labelPadding: 2
            }),
            Cursor.create({
              showTime: true,
              height: '50px',
              opacity: 1,
              customShowTimeStyle: {
                'background-color': '#000',
                color: '#fff',
                padding: '2px',
                'font-size': '10px'
              }
            })
          ]
        })
        /** 加载音频 */
        _this.wavesurfer.load(_this.url)
        // _this.wavesurfer.load(url)
        _this.wavesurfer.on('waveform-ready', () => {
          _this.wavesurfer.zoom(50)

          if (status === 1) { // 已标注
            _this.addRegions(data)
            /** * 初次加载列表 */
            _this.regionList = Object.values(_this.wavesurfer.regions.list)
            _this.$emit('changeRegionList', _this.regionList)
          }
          _this.loading = false
          _this.$emit('sliptLoading')
        })

        /** audioprocess，获取当前时间 */
        _this.wavesurfer.on('audioprocess', function(e) {
          _this.currentTime = _this.changeTime(_this.wavesurfer.getCurrentTime())
        })

        /** 播放当前区域时颜色区分*/
        _this.wavesurfer.on('region-in', function(region) {
          region.update({ color: 'hsla(400, 100%, 30%, 0.5)' })
          _this.wavesurfer.on('region-in', (newRegion) => {
            if (region.id !== newRegion.id) {
              region.update({ color: 'rgba(0,0,0,0.1)' })
            }
          })
        })

        /** 点击区域*/
        _this.wavesurfer.on('region-click', function(region, mouseEvent) {
          region.play() // 播放当前区域，另一种播放方式： _this.wavesurfer.play(start, end)
          _this.$emit('region-click', region)
        })

        if (status === 0) {
          /** 新增区域*/
          _this.wavesurfer.on('region-created', (region) => {
            _this.$emit('region-click', region)
          })

          /** 新增后*/
          _this.wavesurfer.on('region-update-end', (region) => {
          /** 播放区域的两种方式 */
          // _this.wavesurfer.play(_this.currentRegion.start, _this.currentRegion.end)//①
            region.play()// ②
            _this.$emit('getRegionList', region)

            // 新增区域列表
            _this.regionList = Object.values(_this.wavesurfer.regions.list)
            _this.$emit('changeRegionList', _this.regionList)
          })
        }
      })
    },
    // 加载已标注的区域
    addRegions(datas) {
      if (datas) {
        datas.forEach(item => {
          const { reginId, startTime, endTime, label } = item
          const region = {
            label,
            id: reginId,
            start: startTime,
            end: endTime,
            loop: false,
            color: 'hsla(400, 100%, 30%, 0.5)',
            drag: false
          }
          this.wavesurfer.addRegion(region)
        })
      }
    },
    /** 格式化时间格式 */
    changeTime(seconds) {
      seconds = Number(seconds)
      var minutes = Math.floor(seconds / 60)
      seconds = seconds % 60

      // fill up seconds with zeroes
      var secondsStr = Math.round(seconds).toString()
      secondsStr = seconds.toFixed(2)

      if (minutes > 0) {
        return `${minutes < 10 ? '0' + minutes : minutes}:${seconds < 10 ? '0' + secondsStr : secondsStr}`
      }
      return `00:${seconds < 10 ? '0' + secondsStr : secondsStr}`
    },
    /** 改变播放速度 */
    changeAudioRate(value) {
      this.wavesurfer.setPlaybackRate(Number(value))
    },
    /** 次要标签的数量 */
    _secondaryLabelInterval(pxPerSec) {
      if (pxPerSec >= 20 && pxPerSec < 40) {
        return 12
      } else if (pxPerSec >= 0 && pxPerSec < 20) {
        return 10
      } else {
        return Math.floor(10 / this._timeInterval(pxPerSec))
      }
    },
    /** 主要标签的数量 */
    _primaryLabelInterval(pxPerSec) {
      var retval = 1
      if (pxPerSec >= 100) {
        retval = 2
      } else if (pxPerSec >= 80) {
        retval = 1
      } else if (pxPerSec >= 60) {
        retval = 1
      } else if (pxPerSec >= 40) {
        retval = 5
      } else if (pxPerSec >= 20) {
        retval = 2
      } else {
        // retval = Math.ceil(0.5 / pxPerSec) * 60
        retval = 1
      }
      return retval
    },
    /** @param pxPerSec  返回以分钟为单位的值 */
    _timeInterval(pxPerSec) {
      var retval = 1
      if (pxPerSec >= 100) { // 0.5,1,1.5,2,...,9.5,10
        retval = 0.5
      } else if (pxPerSec >= 80) { // 1,2,...,9,10
        retval = 1
      } else if (pxPerSec >= 60) { // 2,4,6,8,10
        retval = 2
      } else if (pxPerSec >= 40) { // 5,10
        retval = 1
      } else if (pxPerSec >= 20) {
        retval = 5
      } else {
        retval = Math.ceil(0.5 / pxPerSec) * 60
      }
      return retval
    },
    /** 返回的时间格式 */
    _formatTimeCallback(seconds, pxPerSec) {
      seconds = Number(seconds)
      var minutes = Math.floor(seconds / 60)
      seconds = seconds % 60

      // fill up seconds with zeroes
      var secondsStr = Math.round(seconds).toString()
      if (pxPerSec >= 25 * 10) {
        secondsStr = seconds.toFixed(2)
      } else if (pxPerSec >= 25 * 1) {
        secondsStr = seconds.toFixed(1)
      }

      if (minutes > 0) {
        if (seconds < 10) {
          secondsStr = '0' + secondsStr
        }
        return `${minutes}:${secondsStr}`
      }
      return secondsStr
    },
    play() {
      this.wavesurfer.play()
      // console.log('tag', this.wavesurfer.getCurrentTime())
    },
    rePlay() {
      this.wavesurfer.play(0)
    },
    pause() {
      this.wavesurfer.pause()
    },
    stop() {
      this.wavesurfer.stop()
    },
    /** 改变音量 */
    changeRound() {
      this.wavesurfer.setVolume(Number(this.sliderArr[1].model * 0.01))
    },
    /** 改变声波大小 */
    changeZoom() {
      this.wavesurfer.zoom(Number(this.sliderArr[0].model))
    }

  }
}
</script>

<style lang="scss" scoped>
.audio{
  background-color:rgba(215, 243, 243, 0.2);
  padding: 0 10px;
  height:230px;
  border:1px solid lightgray;
}
.no-data{
  margin:100px auto;
  // width:50%;
  text-align: center;

}
#timeline{
  height:20px;
  // background-color: cornflowerblue;
}
#waveform{
  position: relative;
  height:128px;
  // background-color: cornflowerblue;
}

.button{
  border:1px solid lightgray;
}
.audio-warp{
  // width:90%;
  // margin:0 auto;

  .button-warp{
    // background-color: rosybrown;
    text-align:center;
    margin: 20px 0 20px 0;
    max-width:400px;
    min-width:180px;
    float:left;
    position: relative;
    p{
      font-size: 1.4em;
      line-height: 30px;
      margin:0;
    }
    img{
      // width:15px;
      // height:15px;
      position: absolute;
      top:10px;
    }
    .svg-icon{
      position: absolute;
      top:12px;
    }
    .el-slider{
      width:100px;
      display: inline-block;
      margin:0  10px 0 25px ;
    }

  }
  .speed{
    span{
      background-color: white;
      padding: 5px 10px;
      border: 1px solid blue;
    }
  }
}

</style>
