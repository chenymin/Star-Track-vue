<template>
  <ion-card>
    <ion-grid no-padding>
      <ion-row align-items-center>
        <ion-col col-12 col-md-5>
          <div v-bind:class="{
            '': !isPlaying,
            'playing': isPlaying ,
            'is-seeking': isSeeking
          }" class="track-img">
            <img :src="fullImage()" alt="">
          </div>
        </ion-col>
        <ion-col>
          <ion-card-content>
            <ion-card-title>
              <h2>{{track.trackName}}</h2>
            </ion-card-title>
            <h3>by {{track.artistName}}</h3>
            <em>{{track.collectionName}}</em>

            <ion-grid>
              <ion-row>
                <ion-col col-3>
                  <ion-fab-button mini @click="toggleSong()" color="light">
                    <ion-icon color="success" :name="isPlaying ? 'square' : 'play'"></ion-icon>
                  </ion-fab-button>
                </ion-col>
                <ion-col col-9>
                  <ion-range :value="progress" max="100" @ionFocus="handleDrag()" @ionBlur="seek($event.target.value)" color="success"></ion-range>
                </ion-col>
              </ion-row>
            </ion-grid>
          </ion-card-content>
          <ion-item>
            <p>Track Duration</p>
            <ion-badge slot="end" color="light">
              {{track.trackTimeMillis | msToMins}}
            </ion-badge>
          </ion-item>

        </ion-col>
      </ion-row>
    </ion-grid>
  </ion-card>
</template>
<style>

  .track-img {
    transition: all 500ms cubic-bezier(0.42, 0, 0, 1.93);
    transform: scale3d(0.8, 0.8, 0.8);
    max-width: 400px;
    margin: auto;
    border-radius: 15px;
    }
    .track-img.playing {
      transform: scale3d(0.89, 0.89, 0.89);
    }

    .track-img.is-seeking {
      transition: all 100ms linear;
      transform: scale3d(0.8, 0.8, 0.8) translateY(-35px);
    }

    .track-img img {
      border-radius: 15px;
    }

  .track-img .background-shadow {
    height: 100%;
    top: 0;
    bottom: 0;
    left: 0;
    position: absolute;
    width: 100%;
    transition: all 300ms cubic-bezier(0.42, 0, 0.36, 1.46);
    z-index: -1;
    opacity: 0;
    background-color: rgb(0, 0, 0);
    filter: blur(20px);
    border-radius: 15px;
    transform: scale3d(0.84, 0.84, 0.84);
  }
  .track-img.playing .background-shadow {
    transform: scale3d(0.9, 0.9, 0.9) translate3d(0, 35px, 0);
  }
  .range-ios .range-knob {
    transition: transform 100ms linear;
    transform: scale3d(0.5, 0.5, 0.5);
    background: map-get($colors, secondary);
  }

  .range-knob-handle.range-knob-pressed .range-knob {
    transform: scale3d(0.8, 0.8, 0.8);
  }
</style>
<script lang="ts">
import { Howl } from "howler";
import { Component, Prop, Vue, Watch } from "vue-property-decorator";

@Component
export default class MusicCard extends Vue {
  @Prop() private track: any;

  // @Prop() private player: Howl;

  @Watch("track")
  trackDidChange() {
  }
  progress: number;
  animationQ: any;
  isPlaying: boolean;
  isFavorite: boolean;
  isSeeking: boolean;
  player: any;
  constructor() {
    super();
    this.progress = 0;
    this.isPlaying = false;
    this.isFavorite = false;
    this.isSeeking = false;
    this.player = null;


  }
  setProgress() {
    if (!this.isSeeking) {
      let seek: any = this.player.seek();
      this.progress = seek / this.player.duration() * 100 || 0;
      requestAnimationFrame(this.setProgress.bind(this));
    }
  }
  playerEnded() {
    this.isPlaying = false;
  }

  fullImage() {
    if (!!this.track) {
      return this.track.artworkUrl100.replace(/100x100bb/, "400x400bb");
    }
  }

  toggleSong() {
    if (this.isPlaying) {
      this.stopSong();
    } else {
      this.playSong();
    }
  }
  playSong() {
    this.player.play();
    this.isPlaying = true;
  }
  stopSong() {
    this.isPlaying = false;
    this.player.stop();
    window.cancelAnimationFrame(this.animationQ);
  }
  seek(val: any) {
    this.isSeeking = false;
    let duration = this.player.duration();
    this.player.seek(duration * (val / 100));
    requestAnimationFrame(this.setProgress.bind(this));
  }
  handleDrag() {
    this.isSeeking = true;
  }
  mounted(){
    this.player = new Howl({
      src: [this.track.previewUrl],
      onplay: this.setProgress.bind(this),
      onend: this.playerEnded.bind(this)
    });
  }
}
</script>
