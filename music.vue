<template>
    <div>
        <div class="audio_box">

            <div class="play">
                <el-popover placement="top" trigger="hover">
                    <div style="text-align: center">
                        <el-progress color="#67C23A" type="circle" :percentage="music.volume"></el-progress>
                        <br>
                        <el-button @click="changeVolume(-10)" icon="el-icon-minus" circle></el-button>
                        <el-button @click="changeVolume(10)" icon="el-icon-plus" circle></el-button>

                    </div>
                    <el-button @click="play" id="play" slot="reference" :icon="music.isPlay?'el-icon-video-pause':'el-icon-caret-right'" circle></el-button>
                </el-popover>
            </div>

            <div class="sliders">
                <el-slider v-model="music.currentTime" :max="music.maxTime" @change="changeTime"></el-slider>
                <audio ref="music">
                    <source :src="musicUrl" type="audio/mpeg">
                </audio>
            </div>

            <div class="time">
                {{cTime}}/{{dTime}}
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: ['musicUrl'],
    data() {
        return {
            music: {
                isPlay: false,
                currentTime: 0,
                maxTime: 0,
                volume: 100
            },
            dTime: "",
            cTime: "00:00"
        }
    },
    mounted() {
        const music = this.$refs.music  // 音频所在对象
        music.addEventListener('canplay', () => {
            const musicTime = music.duration // 获得音频时长
            console.log(musicTime)
            this.music.maxTime = musicTime
            const branch = Math.floor(musicTime / 60) // 计算音频分钟
            const second = Math.ceil(musicTime % 60) // 计算音频秒
            if (branch < 10 && second < 10) { // 四种情况判断音频总时间
                this.dTime = `0${branch}:0${second}`
            } else if (branch < 10) {
                this.dTime = `0${branch}:${second}`
            } else if (second < 10) {
                this.dTime = `${branch}:0${second}`
            } else {
                this.dTime = `${branch}:${second}`
            }


        })
        // 获得音频正在播放时的处理
        music.addEventListener('timeupdate', () => {
            const musicTime = music.duration // 获得音频时长
            const stopTime = music.currentTime // 获得已播放的音频时长
            this.music.currentTime = stopTime
            const branch = Math.floor(stopTime / 60) // 计算已播放的音频分钟
            const second = Math.floor(stopTime % 60) // 计算已播放的音频秒
            if (branch < 10 && second < 10) { // 四种情况判断显示音频以播放时间
                this.cTime = `0${branch}:0${second}`
            } else if (branch < 10) {
                this.cTime = `0${branch}:${second}`
            } else if (second < 10) {
                this.cTime = `${branch}:0${second}`
            } else {
                this.cTime = `${branch}:${second}`
            }
        })
        music.addEventListener("pause",
            () => { //监听暂停
                this.cTime = this.dTime
                this.music.isPlay = false
            }, false);
    },
    methods: {
        play() {
            // 获取所有audios
            var audios = document.getElementsByTagName("audio");
            // 暂停函数
            function pauseAll() {
                var self = this;
                [].forEach.call(audios, function (i) {
                    // 将audios中其他的audio全部暂停
                    i !== self && i.pause();
                })
            }
            // 给play事件绑定暂停函数
            [].forEach.call(audios, function (i) {
                i.addEventListener("play", pauseAll.bind(i));
            })
            if (this.$refs.music.paused) {
                this.$refs.music.play()
            } else {
                this.$refs.music.pause()
            }
            this.music.isPlay = !this.$refs.music.paused
        },
        changeTime(time) {
            this.$refs.music.currentTime = time
            let audio = this.$refs.music
            audio.currentTime = time
        },
        changeVolume(v) {
            this.music.volume += v
            if (this.music.volume > 100) {
                this.music.volume = 100
            }
            if (this.music.volume < 0) {
                this.music.volume = 0
            }
            this.$refs.music.volume = this.music.volume / 100
        }
    }
}
</script>

<style scoped>
.audio_box {
  width: 100%;
  display: flex;
  justify-content: space-around;
  align-items: center;
}
.play {
  flex: 2;
}
.sliders {
  flex: 5;
  margin: 0 5px;
}
.time {
  flex: 3;
}
</style>