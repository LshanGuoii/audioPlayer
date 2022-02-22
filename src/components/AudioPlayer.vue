<!--
 - 自定义音频播放器
 - author: luoshang
-->

<script>
export default {
  name: 'AudioPlayer',
  components: {
    Play: {
      render(h) {
        return !this.$parent.playing ? (
          <div class='play red' icon-class='play' onClick={() => this.$parent.play()} />
        ) : (
          <div class='pause red' icon-class='pause' onClick={() => this.$parent.pause()} />
        )
      }
    },

    Times: {
      render(h) {
        const { parseTime, currentTime, duration } = this.$parent

        return <span class='time'>{`${parseTime(currentTime)} / ${parseTime(duration)}`}</span>
      }
    },

    Progress: {
      mounted() {
        this.$el.querySelector('.el-slider__runway').addEventListener('click', this.handleClick, false)
      },
      beforeDestroy() {
        this.$el.querySelector('.el-slider__runway').removeEventListener('click', this.handleClick, false)
      },
      methods: {
        // 加点击事件解决点击后从 0 开始的问题
        handleClick() {
          this.handleChange(this.val)
        },
        handleChange(val) {
          // 值改变时
          console.log('[ val ]', val)
          if (!this.$refs.progress.$refs.button1.dragging) {
            this.$parent.play()
          }
          this.$parent.setCurrentTime((val / 100) * this.$parent.duration)
        },
        handleInput(val) {
          console.log('[ val222 ]', val)
          // 数据改变时
          this.val = val
          if (!this.$refs.progress.$refs.button1.dragging) return
          this.$parent[this.$refs.progress.$refs.button1.dragging ? 'pause' : 'play']()
          this.$parent.setCurrentTime((val / 100) * this.$parent.duration)
        }
      },
      render(h) {
        return <el-slider ref='progress' value={this.$parent.progress} show-tooltip={false} onChange={evt => this.handleChange(evt)} onInput={evt => this.handleInput(evt)} />
      }
      // methods: {
      //   // 值改变
      //   handleChange(val) {
      //     console.log('[ val1 ]', val)
      //     // console.log('1', this.$refs.progress.$refs.button1.dragging)
      //     if (!this.$refs.progress.$refs.button1.draggging) {
      //       this.$parent.play()
      //     }
      //     this.$parent.setCurrentTime((val / 100) * this.$parent.duration)
      //     console.log('[ this.$parent.duration ]', this.$parent.duration)
      //   },
      //   // 数据改变
      //   handleInput(val) {
      //     console.log('[ val2 ]', val)
      //     // console.log('2', this.$refs.progress.$refs.button1.dragging)
      //     if (!this.$refs.progress.$refs.button1.dragging) return
      //     this.$parent[this.$refs.progress.$refs.button1.dragging ? 'pause' : 'play']()
      //     console.log('xxii')

      //     this.$parent.setCurrentTime((val / 100) * this.$parent.duration)
      //   }
      // },
      // render(h) {
      //   return <el-slider ref='progress' value={this.$parent.progress} show-tooltip={false} onChange={evt => this.handleChange(evt)} onInput={evt => this.handleInput(evt)} />
      // }
    },

    CircleProgress: {
      render(h) {
        const { parseTime, currentTime, duration, percentage } = this.$parent

        return (
          <el-progress type='circle' percentage={percentage} stroke-width={4}>
            <template slot='text'>
              <span class='time'>{`${parseTime(duration - currentTime)}`}</span>
            </template>
          </el-progress>
        )
      }
    },

    Sound: {
      render(h) {
        return (
          <div class='sound-control'>
            {this.$parent.isMuted ? (
              <div class='red' icon-class='mute' onClick={() => this.$parent.handleMute(false)} />
            ) : (
              <div class='red' icon-class='sound' onClick={() => this.$parent.handleMute(true)} />
            )}
            <el-slider
              value={this.$parent.volume}
              min={0}
              max={1}
              step={0.01}
              show-tooltip={false}
              onChange={evt => this.$parent.setVolume(evt)}
              onInput={evt => this.$parent.setVolume(evt)}
            />
          </div>
        )
      }
    },

    HoverSound: {
      render(h) {
        return (
          <div class='hover-sound-control'>
            <el-slider
              value={this.$parent.volume}
              min={0}
              max={1}
              step={0.01}
              show-tooltip={false}
              onChange={evt => this.$parent.setVolume(evt)}
              onInput={evt => this.$parent.setVolume(evt)}
            />
            {this.$parent.isMuted ? (
              <div class='red' icon-class='mute' onClick={() => this.$parent.handleMute(false)} />
            ) : (
              <div class='red' icon-class='sound' onClick={() => this.$parent.handleMute(true)} />
            )}
          </div>
        )
      }
    },

    Download: {
      render(h) {
        return <div class='red' class='download' icon-class='download' onClick={() => this.$parent.handleDownload()} />
      }
    }
  },
  props: {
    src: {
      type: String,
      required: true
    },
    type: {
      type: String,
      default: 'simple',
      validator: val => ['gray', 'white', 'simple', 'circle'].includes(val)
    },
    // play-播放按钮, times-时间, progress-播放进度, hoverSound-音量控制（hover版）, download-下载按钮
    // sound-音量控制（音量条常驻版）
    layout: {
      type: String,
      default: 'play, times, progress, hoverSound, download'
    }
  },
  data() {
    return {
      duration: 0,
      currentTime: 0,
      progress: 0,

      volume: 0.5,

      playing: false,
      isMuted: false
    }
  },
  computed: {
    percentage() {
      return this.duration > 0 ? 100 - (this.currentTime / this.duration) * 100 : 0
    }
  },
  mounted() {
    this.setVolume(this.volume)
  },
  methods: {
    parseTime(val) {
      val = Math.round(val)
      let m = 0
      let s = val
      if (val > 60) {
        m = parseInt(val / 60)
        s = val - 60 * m
      }

      return `${(m + '').padStart(2, '0')}:${(s + '').padStart(2, '0')}`
    },

    // 开始请求数据
    handleLoadstart() {
      this.$emit('loadstart')
    },
    // 正在请求数据，缓冲
    handleProgress() {
      if (!this.$refs.audio) {
        return
      }
      this.duration = this.$refs.audio.duration
      this.$emit('progress')
    },
    // 单前播放时间发生改变
    handleTimeupdate() {
      if (!this.$refs.audio) {
        return
      }
      this.currentTime = this.$refs.audio && this.$refs.audio.currentTime
      this.progress = parseInt((this.currentTime / this.duration) * 100)
      console.log('[ this.progress ]', this.progress)

      this.$emit('timeupdate', { currentTime: this.currentTime, duration: this.duration })
    },
    // 音频已完全载入
    handleCanplaythrough() {
      this.duration = this.$refs.audio.duration
      this.$emit('canplaythrough')
    },
    // 音频已缓冲至可播放状态
    handleCanplay() {
      this.duration = this.$refs.audio && this.$refs.audio.duration
      this.$emit('canplay', { duration: this.duration })
    },
    // 播放回调
    handlePlay() {
      this.playing = true
      this.$emit('play')
    },
    // 暂停回调
    handlePause() {
      this.playing = false
      this.$emit('pause')
    },
    // 已播完
    handleEnded() {
      this.playing = false
      this.$emit('ended')
    },

    play() {
      if (this.$refs.audio) {
        this.$refs.audio.play()
      }
    },
    pause() {
      if (this.$refs.audio) {
        this.$refs.audio.pause()
      }
    },

    setCurrentTime(time) {
      console.log('[ time ]', time)
      if (this.$refs.audio) {
        console.log('[ this.$refs.audio ]', this.$refs.audio.currentTime)
        this.$refs.audio.currentTime = time
        console.log('[ this.$refs.audio.currentTime ]', this.$refs.audio.currentTime)
      }
    },
    setVolume(volume) {
      if (this.$refs.audio) {
        this.isMuted = volume === 0
        this.$refs.audio.volume = volume
        this.volume = volume
      }
    },
    handleMute(isMuted) {
      if (!this.$refs.audio) {
        return
      }
      if (isMuted) {
        this.tempVolume = this.$refs.audio.volume
      }
      this.isMuted = isMuted
      this.volume = isMuted ? 0 : this.tempVolume
      this.setVolume(this.volume)
    },

    handleDownload() {
      fetch(this.src)
        .then(res => res.blob())
        .then(blob => {
          const a = document.createElement('a')
          document.body.appendChild(a)
          a.style.display = 'none'
          // 使用获取到的blob对象创建的url
          const url = window.URL.createObjectURL(blob)
          a.href = url
          // 指定下载的文件名
          a.download = this.src.split('/').pop()
          a.click()
          document.body.removeChild(a)
          // 移除blob对象的url
          window.URL.revokeObjectURL(url)
        })
    }
  },
  render(h) {
    const layout = this.layout
    if (!layout) return null

    const template = <div class='audio-player-wrap'></div>
    const innerTemplate = <div class={['audio-player', this.type && `audio-player-${this.type}`]}></div>
    const audioTemplate = (
      <audio
        ref='audio'
        class='audio'
        controls
        src={this.src}
        onLoadstart={evt => this.handleLoadstart(evt)}
        onProgress={evt => this.handleProgress(evt)}
        onTimeupdate={evt => this.handleTimeupdate(evt)}
        onCanplaythrough={evt => this.handleCanplaythrough(evt)}
        onCanplay={evt => this.handleCanplay(evt)}
        onPlay={evt => this.handlePlay(evt)}
        onPause={evt => this.handlePause(evt)}
        onEnded={evt => this.handleEnded(evt)}>
        您的浏览器不支持 audio 元素。
      </audio>
    )

    if (this.type === 'circle') {
      innerTemplate.children = [<CircleProgress />]
    } else {
      innerTemplate.children = template.children || []

      const TEMPLATE_MAP = {
        play: <Play />,
        times: <Times />,
        progress: <Progress />,
        sound: <Sound />,
        hoverSound: <HoverSound />,
        download: <Download />,
        slot: <slot>{this.$scopedSlots.default ? this.$scopedSlots.default(this) : ''}</slot>
      }
      const components = layout.split(',').map(item => item.trim())

      components.forEach(compo => {
        innerTemplate.children.push(TEMPLATE_MAP[compo])
      })
    }

    template.children = [audioTemplate, innerTemplate]

    return template
  }
}
</script>

<style lang="scss">
.red {
  background: red;
  width: 10px;
  height: 10px;
}
.audio-player-wrap {
  display: inline-block;
  .audio {
    display: none;
  }
  .audio-player {
    display: flex;
    align-items: center;
    &.audio-player-gray,
    &.audio-player-white {
      border-radius: 16px;
      padding: 7px 18px;
    }
    &.audio-player-gray {
      background: #f7f8fa;
    }
    &.audio-player-white {
      background: #fefffe;
      box-shadow: 0px 2px 14px 1px rgba(231, 233, 237, 0.65);
      oo .hover-sound-control {
        &::before {
          background: #fefffe;
        }
        &::after {
          background: #f7f8fa;
        }
      }
    }

    &.audio-player-simple {
      background: #fefffe;
      .hover-sound-control {
        &::before {
          background: #fff;
        }
        &::after {
          background: #f7f8fa;
        }
      }
    }

    > * {
      margin-left: 12px;
      &:first-child {
        margin-left: 0;
      }
      &:last-child {
        margin-left: 16px;
      }
    }
  }
  .audio-player-simple {
    .sound {
      font-size: 16px;
    }
  }
  .svg-icon {
    cursor: pointer;
  }

  .time {
    display: inline-block;
    min-width: 94px;
    text-align: right;
  }
  .el-progress--circle {
    .time {
      text-align: center;
      min-width: 0;
    }
  }

  .hover-sound-control {
    position: relative;
    display: flex;
    align-items: center;

    &::before,
    &::after {
      content: '';
      position: absolute;
      left: -8px;
      top: -4px;
      right: -8px;
      bottom: -4px;
      background: #eff0f2;
      border-radius: 12px;
      z-index: 1;
      transition: left 0.5s, opacity 0.5s;
      opacity: 0;
    }
    &::before {
      background: #f7f8fa;
      opacity: 0;
    }
    .svg-icon {
      position: relative;
      z-index: 2;
    }
    .el-slider {
      position: absolute;
      right: 24px;
      width: 0;
      transition: width 0.5s;
      overflow: hidden;
      margin: 0;
      z-index: 2;
    }
    &:hover {
      &::before {
        left: -76px;
        opacity: 1;
      }
      &::after {
        left: -64px;
        opacity: 1;
      }
      .el-slider {
        width: 42px;
        overflow: visible;
      }
    }
  }

  .sound-control {
    position: relative;
    display: flex;
    align-items: center;

    .el-slider {
      width: 104px;
    }
  }

  .el-slider {
    width: 82px;
    margin: 0 4px 0 16px;
  }
  .el-slider__runway {
    height: 2px;
    margin: 8px 0;
    background-color: #c0c1c6;
  }
  .el-slider__bar {
    height: 2px;
  }
  .el-slider__button-wrapper {
    width: 10px;
    height: 10px;
    top: 50%;
    margin-top: -5px;
    display: flex;
    align-items: center;
    z-index: 1;
    &:hover {
      cursor: pointer;
    }
  }
  .el-slider__button {
    width: 10px;
    height: 10px;
    background: #fff;
    box-shadow: 0px 0px 1px 0px rgba(0, 0, 0, 1);
    border: 0;
    transition: none;
    &:hover,
    &.hover,
    &.dragging {
      transform: scale(1);
      cursor: pointer;
    }
  }
  .el-progress--circle {
    margin: 0;
    .el-progress-circle {
      box-shadow: 0px 2px 14px 1px rgba(231, 233, 237, 0.65);
      border-radius: 50%;
    }
    .el-progress-circle__track {
      stroke: none;
    }
    .el-progress__text {
      font-size: 24px;
      font-weight: 500;
      color: #3d3d3e;
      line-height: 33px;
    }
  }
}
</style>
