<template lang="pug">
.now-playing
  transition(name="fadeslide")
    img(v-if="image.length > 0 && (albumArtVisible || alwaysShowAlbumArt) && !settingsVisible && !toast.visible && !hideAll" :src="image")
  transition(name="fade")
    div(:class="{ transitioning, initialized }" v-if="!settingsVisible && !toast.visible && !hideAll && (trackInfoVisible || alwaysShowTrackInfo)")
      span.name
        span(v-if="name" :style="{ transition: `opacity ${colorTransitionDuration}ms` }") {{ name }}
      br
      span.artists
        span(v-if="artists" :style="{ transition: `opacity ${colorTransitionDuration}ms` }") {{ artists }}
</template>

<script>
import { mapState} from 'vuex' 
import { SET_ALBUM_ART_VISIBLE, SET_TRACK_INFO_VISIBLE } from '@/vuex/mutation-types'

export default {
  data () {
    return {
      delay: 4000, // Visibility duration (ms)
      transitioning: false,
      initialized: false,
      transitionendIndex: 0,
      image: '',
      name: '',
      artists: ''
    }
  },
  computed: {
    ...mapState([
      'color', 
      'colorTransitionDuration', 
      'currentlyPlaying', 
      'albumArtVisible', 
      'alwaysShowAlbumArt', 
      'trackInfoVisible', 
      'alwaysShowTrackInfo',
      'toast',
      'settingsVisible',
      'hover',
      'hideAll'
    ])
  },
  watch: {
    currentlyPlaying: {
      handler (val, old) {
        const values = this.getCurrentlyPlaying(val)
        this.image = values.image
        this.artists = values.artists
        this.name = values.name
        this.show(val, old)
        // this.transition(values)
      },
      immediate: true
    }
  },
  methods: {
    show (val, old) {
      this.$store.commit(SET_ALBUM_ART_VISIBLE, true)
      this.$store.commit(SET_TRACK_INFO_VISIBLE, true)
    

      if (!old) return
      
      this.timeout = setTimeout(() => {
        if (this.alwaysShowAlbumArt === false) {
          this.$store.commit(SET_ALBUM_ART_VISIBLE, false)
        }

        if (this.alwaysShowTrackInfo === false) {
          this.$store.commit(SET_TRACK_INFO_VISIBLE, false)
        }
      }, this.delay)
    },
    getCurrentlyPlaying (val = this.currentlyPlaying) {
      const name = val.name || false
      const album = val.album 
      const image = album ? album.images[1].url : false
      const artists = val.artists ? val.artists.map(artist => artist.name).join(', ') : ''
      return { name, image, artists }
    }
  }
}
</script>

<style lang="scss" scoped>
.now-playing {
  @include position(fixed, null 0 0 0);
  @include flex(center, flex-start);
  @include scale(min-height 210px 140px);
  padding: 30px;
  z-index: 200;
  text-align: left;
}

img {
  @include scale(width 150px 80px, height 150px 80px);
  margin-right: 15px;
}

.name {
  @include share;
  @include scale(font-size 32px 18px);
}

.artists {
  @include scale(font-size 18px 14px);
}

.name, .artists {
  position: relative;
  span {
  background: black;
  color: white;
  padding: 3px 6px;
    position: relative;
    z-index: 10;

    // opacity: 0;
  }
}

i {
  @include position(absolute, 0 0 0 0);
  transform: scaleX(0);
  z-index: 20;
  display: block;
  background: #ffffff;
  transform-origin: left;
}

.initialized {
  .name, .artists {
    span { opacity: 1; }
  }
}

.transitioning i { transform: scaleX(1); }
</style>
