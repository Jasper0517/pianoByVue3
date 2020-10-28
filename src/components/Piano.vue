<template>
  <h1>Piano by Vue3</h1>
  <audio v-for="pack in soundPackIndex" :ref="el => { els[pack] = el }" :key="pack">
    <source :src="`https://awiclass.monoame.com/pianosound/set/${pack}.wav`" type="audio/wav"/>
  </audio>
  <div class="piano">
    <div v-for="(key,index) in displayKey" :key="index" class="piano-key">
      <div
        v-if="key.type === 'white'"
        class="white"
        :class="{'playing' : (key.num === nowPlayingKey) ? true : false}"
        @click="addNote(key.num)"
      >
        <div class="key">{{ String.fromCharCode(key.key) }}</div>
      </div>
      <div v-else class="black" :class="{'playing' : (key.num === nowPlayingKey) ? true : false}" @click="addNote(key.num)">
        <div class="key">{{ String.fromCharCode(key.key) }}</div>
      </div>
    </div>
  </div>

  <div class="buttons">
    <span class="button" @click="getSample">Get Sample</span>
    <span v-if="!isPlaying" class="button" @click="startPlay">Start Play</span>
    <span v-else class="button" @click="stopPlay"> StopPlay</span>
    <span v-if="!isRecording" class="button" @click="startRecord">Start Record</span>
    <span v-else class="button" @click="stopRecord">Stop Record</span>
    <span class="button" @click="clearSong">Clear Song</span>
  </div>
  <div class="play-time">{{ playTime }}ms</div>

  <ul class="notes">
    <li
      v-for="(songKey, index) in song"
      :key="index"
      class="note"
      :class="{'playing': (index == nowPlayingNote - 1) &amp;&amp; isPlaying ? true : false}"
      @click="playNote(songKey.num)"
    >
      <div class="num">{{ songKey.num }} </div>
      <div class="time">{{ songKey.time }}</div>
    </li>
  </ul>
</template>

<script>

import { ref, onMounted } from 'vue'

import axios from 'axios'

import {
  SOUND_PACK_INDEX,
  DISPLAY_INDEX,
  DEFAULT_SONG
} from '../constant/index.js'

export default {
  name: 'Piano',
  setup() {
    // state
    const soundPackIndex = ref([])
    const displayKey = ref([])
    const song = ref([])
    const nowPlayingKey = ref(0)
    const nowPlayingNote = ref(0)
    const isRecording = ref(false)
    const isPlaying = ref(false)
    const playTime = ref(0)
    const playInterval = ref(0)

    // html ref
    const els = ref([])
    soundPackIndex.value = SOUND_PACK_INDEX
    displayKey.value = DISPLAY_INDEX
    song.value = DEFAULT_SONG

    onMounted(() => {
      window.onkeydown = e => {
        const isKeyExists = displayKey.value.find(key => key.key === e.keyCode)
        if (isKeyExists) {
          addNote(isKeyExists.num)
          nowPlayingKey.value = isKeyExists.num
        }
      }

      window.onkeyup = e => { nowPlayingKey.value = 0 }
    })

    const playNote = key => {
      const audio = els.value[key]
      audio.currentTime = 0
      audio.play()
    }

    const addNote = key => {
      playNote(key)
      if (!isRecording.value) return
      song.value.push({
        'num': key,
        'time': playTime.value
      })
    }

    const getSample = async() => {
      try {
        const { data } = await axios.get('https://awiclass.monoame.com/api/command.php?type=get&name=music_dodoro')
        song.value = data
      } catch (error) {
        console.log(`Get Song Error: ${error}`)
      }
    }

    const stopPlay = () => {
      clearInterval(playInterval.value)
      isPlaying.value = false
      playTime.value = 0
      nowPlayingKey.value = NaN
      nowPlayingNote.value = 0
    }

    const startPlay = () => {
      if (isRecording.value) {
        console.error('錄音中無法播放')
        return
      }
      if (song.value.length <= 0) {
        console.error('無樂譜無法播放')
        return
      }

      isPlaying.value = true

      const songTimeList = song.value.map(key => key.time)
      playInterval.value = setInterval(() => {
        playTime.value++
        if (playTime.value - songTimeList[songTimeList.length - 1] >= 100) stopPlay()
        for (let i = 0; i < songTimeList.length; i++) {
          if (songTimeList[i] === playTime.value) {
            const num = song.value[i].num
            nowPlayingKey.value = num
            nowPlayingNote.value = i + 1
            playNote(num)
          }
        }
      }, 1)
    }

    const stopRecord = () => {
      isRecording.value = false
      playTime.value = 0
      clearInterval(playInterval.value)
    }

    const startRecord = () => {
      isRecording.value = true
      song.value = []
      playInterval.value = setInterval(() => {
        playTime.value++
      }, 1)
    }

    const clearSong = () => {
      if (isRecording.value) return
      song.value = []
    }

    return {
      // state
      soundPackIndex,
      displayKey,
      song,
      nowPlayingKey,
      nowPlayingNote,
      isRecording,
      isPlaying,
      playTime,
      els,
      // function
      addNote,
      playNote,
      getSample,
      startPlay,
      stopPlay,
      startRecord,
      stopRecord,
      clearSong
    }
  }
}
</script>

<style lang="stylus" scoped>

  .piano
    margin-bottom 50px
    display inline-block
    box-shadow 0px 0px 40px -5px rgba(black, .400)

  .center
    position relative
    width 100%
    margin-top 100px
    text-align center

  .piano-key
    display inline-block
    position relative
    cursor pointer
    .key
      position absolute
      bottom -25px
      left 50%
      transform translateX(-50%)
      font-size 10px
      color #eee

  .black
    top 0px
    position absolute
    width 22px
    height 166px
    margin-left -11px
    margin-right -11px
    border 1px solid black
    z-index 20
    background-color lighten(black, 30)
    transform translate(-3px,-3px)
    &:hover
      transform translate(0px,0px)
      background-color black
    &.playing
      background-color black

  .white
    width 44px
    height 300px
    border 1px solid #eee
    transform translate(-3px,-3px)
    &:hover
      transform translate(0px,0px)
      background-color darken(#eee, 5)
    &.playing
      background-color darken(#eee, 5)

  .play-time
    margin-top 20px

  ul
    .note
      display inline-block
      border-right 1px solid #585858
      padding 2px 5px
      cursor pointer
      &.playing
        background-color darken(#eee, 5)
      &:hover
        background-color #eee
      .num
        font-size 16px
        color rgba(black,.500)
      .time
        font-size 10px
        color darken(#eee, 10)

  .buttons
    width 50%
    display flex
    justify-content center
    align-items center
    margin 0 auto

  .button
    background-color transparent
    border none
    border 1px solid black
    border-radius 5px
    cursor pointer
    padding 5px 10px
    margin 0 10px
    &:hover
      background-color #585858
      color white

  .notes
    width 70%
    max-width 700px
    margin 0 auto
    text-align left
    padding 20px
    li
      width 30px
      margin 2px 0px
</style>
