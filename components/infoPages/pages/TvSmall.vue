<template>
  <div class="background">
    <!-- Dynamic banner image -->
    <v-img :src="`https://image.tmdb.org/t/p/original${storeTv.info.backdrop_path}`" class="backdrop-image">

      <!-- Poster image-->
      <v-container fluid class="overlay-container">
        <v-row align="center">
          <MediaPoster v-if="storeTv.info" class="pl-6"/>
        </v-row>
      </v-container>
    </v-img>
            
    <!-- Title and Release Date -->
    <v-container>
      <div class="tv-title-div">
        <span class="tv-title">{{storeTv.info.name}}<span v-if="storeTv.info.first_air_date" class="released-year">({{ storeTv.info.first_air_date | formatYear }})</span></span>
      </div>

      <!-- Rating and Trailer Button -->
      <v-row v-if="storeTv.info" align="center" justify="space-around" class="pl-4 pr-4">
        <div class="rating-div">
          <PercentageWheel v-if="storeTv.info.vote_average" class="mt-3" :rating="storeTv.info.vote_average" />
          <h3 class="user-score">User Score</h3>
        </div>
        <TrailerDialog v-if="storeTv.trailer" :trailerId="storeTv.trailer.key" class="mt-3 ml-6" />
      </v-row>
    </v-container>
    
    <!-- Genres and Runtime -->
    <div class="tv-info black-background">
      <div v-if="storeTv.info.genres.length" class="tv-info-subdiv">
        <nuxt-link v-for="(genre, i) in genreList" :key="i" :to="{ path: `/list/tv/genres/${genre.id}` }" class="link">
          <span>
            {{genre.formattedName}}
          </span>
        </nuxt-link>
      </div>
      <span v-if="storeTv.info.episode_run_time.length && storeTv.info.genres.length" class="bullet-divider">&#8226;</span>
      <div v-if="storeTv.info.episode_run_time.length" class="tv-info-subdiv">
        <span>{{ storeTv.info.episode_run_time[0] | formatRuntime }}</span>
      </div>
    </div>
        
    <v-container>
      <div class="pl-3 pr-3">

        <!-- Overview -->
        <div v-if="storeTv.info.overview">
          <Overview :overview="storeTv.info.overview" />
        </div>

        <!-- Creator and Crew Links -->
        <v-row v-if="storeTv.crew || storeTv.info.created_by.length">
          <v-col cols="6" md="4" v-for="(creator, i) in storeTv.info.created_by" :key="i">
            <nuxt-link :to="{ path: `/info/people/${creator.id}` }" class="link crew">
              <div class="crew-name">{{creator.name}}</div>
            </nuxt-link>
            <div class="crew-role">Creator</div>
          </v-col>
          <v-col cols="6" md="4" v-if="storeTv.crew.novel.name">
            <nuxt-link :to="{ path: `/info/people/${storeTv.crew.novel.id}` }" class="link crew">
              <div class="crew-name">{{storeTv.crew.novel.name}}</div>
            </nuxt-link>
            <div class="crew-role">Novel</div>
          </v-col>
          <v-col cols="6" md="4" v-if="storeTv.crew.composer.name">
            <nuxt-link :to="{ path: `/info/people/${storeTv.crew.composer.id}` }" class="link crew">
              <div class="crew-name">{{storeTv.crew.composer.name}}</div>
            </nuxt-link>
            <div class="crew-role">Original Music</div>
          </v-col>
        </v-row>
      </div>
    </v-container>
        
    
    <div class="black-background">
      <v-container>
        <!-- Seasons cards -->
        <div v-if="storeTv.seasons.length">
          <v-row>
            <v-col cols="6" sm="4">
              <v-select :items="seasonSelectItems" item-text="name" item-value="index" v-model="season" outlined></v-select>
            </v-col>
          </v-row>
          <EpisodeCardSmall v-for="(episode, i) in storeTv.seasons[season].episodes" :key="i" :episode="episode"/>
        </div>

        <!-- Cast cards -->
        <div v-if="storeTv.cast.length" class="subheading-div">
          <h3 class="info-subheading-description">Meet the stars</h3>
        </div>
        <PersonCarousel v-if="storeTv.cast.length" :useStateCast="true" subheading="true" />

        <!-- Similar Tv cards -->
        <div v-if="storeTv.similarMedia.length">
          <div class="subheading-div">
            <h3 class="info-subheading-description">More like this</h3>
          </div>
          <MediaCarousel :useStateSimilarMedia="true" />
        </div>

      </v-container>

      <BottomBar />
      
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import MediaPoster from '@/components/infoPages/MediaPoster.vue'
import MediaCarousel from '@/components/sliders||carousels/MediaCarousel.vue'
import PercentageWheel from '@/components/infoPages/PercentageWheel.vue'
import TrailerDialog from '@/components/infoPages/TrailerDialog.vue'
import Overview from '@/components/infoPages/Overview.vue'
import PersonCarousel from '@/components/sliders||carousels/PersonCarousel.vue'
import EpisodeCardSmall from '@/components/cards/EpisodeCardSmall.vue'
import BottomBar from '@/components/infoPages/BottomBar.vue'
import FastAverageColor from 'fast-average-color'

export default {
  name: 'TvSmall',
  components: {
    MediaPoster,
    MediaCarousel,
    PercentageWheel,
    TrailerDialog,
    Overview,
    PersonCarousel,
    EpisodeCardSmall,
    BottomBar
  },
  data() {
    return {
      season: 0
    }
  },
  computed: {
    storeTv() {
      return this.$store.state.media.media
    },
    genreList() {
      if (this.storeTv.info && this.storeTv.info.genres.length) {
        let genres = this.storeTv.info.genres
        if (genres.length > 3) genres = genres.slice(0, 3)
        const formattedGenres = genres.map((genre, index) => {
          if (index === genres.length -1) {
            genre["formattedName"] = genre.name
            return genre
          }
          genre["formattedName"] = `${genre.name},`
          return genre
        })
        return formattedGenres
      }
    },
    seasonSelectItems() {
      if (this.storeTv.seasons && this.storeTv.seasons.length) {
        if (this.storeTv.seasons[0].name === 'Specials') this.season = 1
        return this.storeTv.seasons.map((season, index) => {
          return {
            name: season.name,
            index
          }
        })
      }
    }
  }
}
</script>

<style scoped>
.black-background {
  background-color: #0f0f0f
}
.backdrop-image {
  height: 350px;
}
.overlay-container {
  height: 100%;
  width: 100%;
  background: rgb(0,0,0);
  background: linear-gradient(90deg, rgba(0,0,0,1) 0%, rgba(8,7,7,0.9) 20%, rgba(8,7,7,0.85) 23%, rgba(8,7,7,0.8) 26%, rgba(0,0,0,0.05) 45%);
  align-items: center;
  display: flex;
}
.tv-title-div {
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 10px 16px 10px 16px;
}
.tv-title {
  font-size: 1.4em;
  font-weight: 750;
}
.released-year {
  color: lightgrey;
  font-size: 1.1em;
  font-weight: normal;
  padding-left: 6px;
}
.rating-div {
  display:flex;
  align-items: center;
}
.user-score {
  padding-top: 9px;
  padding-left: 14px;
  font-size: 1em;
}
.tv-info {
  margin-bottom: 5px;
  padding-top: 10px;
  padding-bottom: 10px;
  display: flex;
  align-content: center;
  justify-content: center;
  font-size: 0.96em;
  font-weight: 500;
  color: white;
}
.tv-info-subdiv {
  align-self: center;
}
.bullet-divider {
  margin: 2px 12px 0 12px;
  font-size: 1.5em;
}
.link {
  text-decoration: none;
  color: white;
}
.link:hover {
  text-decoration: underline;
}
.link.crew:hover {
  text-decoration: none;
}
.crew-name {
  font-weight: 715;
  font-size: 0.97em;
}
.crew-role {
  font-weight: 400;
  font-size: 0.9em;
  color: lightgrey;
}
/* MEDIA QUERIES */
/* XS */
@media(max-width: 599px) {
  .backdrop-image {
    height: 230px;
  }
  .tv-title {
    font-size: 1.3em;
  }
  .released-year {
    font-size: 1em;
  }
  .tv-info {
    font-size: 0.93em;
    font-weight: 450;
  }
}
</style>