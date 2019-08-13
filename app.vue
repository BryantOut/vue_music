<template>
  <div class="wrapper">
    <div class="demo-title">
      {{currentSong.title}}
    </div>
    <div class="a-player">
      <div class="operators">
        <div class="icon">
          <div class="iconfont icon-prevent" @click.stop.prevent="back"></div>
        </div>
        <div class="icon">
          <div class="iconfont icon-previous" @click.stop.prevent="prev"></div>
        </div>
        <div class="icon">
          <div class="iconfont center" :class="playIcon" @click.stop.prevent="togglePlaying"></div>
        </div>
        <div class="icon">
          <div class="iconfont icon-nextsong" @click.stop.prevent="next"></div>
        </div>
        <div class="icon">
          <div class="iconfont icon-addto" @click.stop.prevent="forward"></div>
        </div>
      </div>
      <div class="progress-wrapper">
        <div class="progress-bar-wrapper">
          <div class="progress-bar" ref="progressBar">
            <div class="bar-inner">
              <div class="progress" ref="progress"></div>
              <div
                class="time"
                @touchstart.prevent="progressTouchStart"
                @touchmove.prevent="progressTouchMove"
                @touchend="progressTouchEnd"
                ref="time">
                {{innerTextTime}}
              </div>
            </div>
          </div>
        </div>
      </div>
      <audio ref="audio" @timeupdate="updateTime"></audio>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'vue-audio',
    data() {
      return {
        playing: false,

        playlist: [
          {
            title: 'EasonChan - 床头床尾',
            url: require('./songs/EasonChan.mp3')
          },
          {
            title: 'Blakey - Addicted',
            url: require('./songs/Blakey - Addicted.mp3')
          }
        ],
        currentIndex: 0,
        count: 0,
        innerTextTime: '0:00/0:00',
      }
    },
    computed: {
      playIcon() {
        return this.playing ? 'icon-play' : 'icon-movie';
      },
      currentSong() {
        return this.playlist[this.currentIndex]
      }
    },
    created() {
      this.touch = {}
      this.$nextTick(() => {
        this.ready()
      })
    },
    methods: {
      ready() {
        this._offset(0)
        this.$refs.audio.src = this.currentSong.url
      },
      // 播放/暂停
      togglePlaying() {
        if (this.$refs.audio.paused) {
          console.log('play', this.playing)
          this.playing = true
          this.$refs.audio.play()
        } else {
          console.log('pause', this.playing)
          this.playing = false
          this.$refs.audio.pause()
        }
      },
      // 下一首
      next() {
        let index = this.currentIndex + 1
        if (index === this.playlist.length) {
          index = 0
        }
        this.currentIndex = index
        this.ready()
        this.playing = true
        this.$refs.audio.play()
      },
      // 上一首
      prev() {
        let index = this.currentIndex - 1;
        if (index === -1) {
          index = this.playlist.length - 1;
        }
        this.currentIndex = index
        this.musicReady()
        this.playing = true
        this.$refs.audio.play()
      },
      // -15s
      back() {
        this.count = this.$refs.audio.currentTime
        this.count -= 15
        if (this.count < 0) {
          this.count = 0
          this.playing = false
          this.$refs.audio.pause()
        } else {
          this.playing = true
          this.$refs.audio.play()
        }
        this.$refs.audio.currentTime = this.count
        console.log(this.count)
      },
      // +15s
      forward() {
        this.count = this.$refs.audio.currentTime
        this.count += 15
        if (this.count > this.$refs.audio.duration) {
          this.count = this.$refs.audio.duration
          this.playing = false
          this.$refs.audio.pause()
        } else {
          this.playing = true
          this.$refs.audio.play()
        }
        this.$refs.audio.currentTime = this.count
        console.log(this.count)
      },
      progressTouchStart(e) {
        this.touch.initiated = true
        this.touch.startX = e.touches[0].pageX
        this.touch.left = this.$refs.progress.clientWidth
        this.playing = false
        this.$refs.audio.pause()
      },
      progressTouchMove(e) {
        if (!this.touch.initiated) {
          return
        }
        const deltaX = e.touches[0].pageX - this.touch.startX
        const offsetWidth = Math.min(this.$refs.progressBar.clientWidth - this.$refs.time.clientWidth, Math.max(0, this.touch.left + deltaX))
        this._offset(offsetWidth)
      },
      progressTouchEnd(e) {
        this.touch.initiated = false
        this._triggerPercent()
        this.playing = true
        this.$refs.audio.play()
      },
      updateTime() {
        this.$refs.time.innerText = `${this.format(this.$refs.audio.currentTime)}/${this.format(this.$refs.audio.duration)}`
        const percent = this.$refs.audio.currentTime / this.$refs.audio.duration
        const barWidth = this.$refs.progressBar.clientWidth - this.$refs.time.clientWidth
        const offsetWidth = percent * barWidth
        this._offset(offsetWidth)
        if (this.$refs.audio.currentTime === this.$refs.audio.duration) {
          this.playing = false
          this.$refs.audio.pause()
        }
      },
      _triggerPercent() {
        const barWidth = this.$refs.progressBar.clientWidth - this.$refs.time.clientWidth
        const percent = this.$refs.progress.clientWidth / barWidth
        this.$refs.audio.currentTime = this.$refs.audio.duration * percent
      },
      _offset(offsetWidth) {
        this.$refs.progress.style.width = `${offsetWidth}px`
        this.$refs.time.style.transform = `translate3d(${offsetWidth}px, 0, 0)`
        this.$refs.time.style.webkitTransform = `translate3d(${offsetWidth}px, 0, 0)`
      },
      format(interval) {
        interval = interval | 0
        const minute = interval / 60 | 0
        const second = this._pad(interval % 60)
        return `${minute}:${second}`
      },
      _pad(num, n = 2) {
        let len = num.toString().length
        while (len < n) {
          num = '0' + num
          len++
        }
        return num
      }

    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" rel="stylesheet/less" type="text/less" scoped>
  @import "./assets/less/index.less";

  .wrapper {
    width: 750px;
    margin: 16px auto;
    background-color: #f2f5f5;
    padding: 16px;
    border-radius: 6px;
  }

  .a-player {
  }

  .operators {
    display: flex;
    margin-bottom: 14px;

    .icon {
      flex: 1;
      text-align: center;
      line-height: 59px;
      height: 59px;

      .iconfont {
        font-size: 32px;
        color: #666;
        font-weight: 500;
      }
    }

    .center {
      display: inline-block;
      width: 59px;
      height: 59px;
      background-size: 59px 59px;
      background-repeat: no-repeat;

      &.play {

      }

      &.pause {

      }
    }
  }

  .progress-wrapper {
    .progress-bar {
      height: 20px;

      .bar-inner {
        position: relative;
        top: 7px;
        height: 4px;
        background: rgba(0, 0, 0, .3);

        .progress {
          position: absolute;
          height: 100%;
          background-color: #3399fe;
        }

        .time {
          position: absolute;
          top: -7px;
          display: inline-block;
          font-size: 12px;
          color: #fff;
          background-color: rgba(0, 0, 0, .9);
          padding: 1px 3px;
          border-radius: 9px;
          cursor: pointer;
        }
      }
    }
  }
</style>
