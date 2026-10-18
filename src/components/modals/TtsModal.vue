<template>
  <v-dialog v-model="ttsModal" max-width="620px">
    <template #activator="{ on, attrs }">
      <v-btn
        :color="$vuetify.theme.dark ? 'grey darken-3' : 'grey lighten-1'"
        class="mb-2"
        v-bind="attrs"
        v-on="on"
      >
        <v-icon left>mdi-text-to-speech</v-icon>
        {{ $t('tts.title') }}
      </v-btn>
    </template>
    <v-card>
      <v-card-title>
        <v-icon left>mdi-text-to-speech</v-icon>
        <span class="text-h5">{{ $t('tts.title') }}</span>
      </v-card-title>
      <v-card-text>
        <v-textarea
          v-model="text"
          :label="$t('tts.placeholder')"
          :counter="800"
          filled
          autofocus
          rows="4"
          prepend-inner-icon="mdi-text"
          @keydown.ctrl.enter="playNow"
        ></v-textarea>

        <v-row>
          <v-col cols="7">
            <v-select
              v-model="language"
              :items="languages"
              :label="$t('tts.language')"
              item-text="name"
              item-value="code"
              hide-details
            ></v-select>
          </v-col>
          <v-col cols="5">
            <v-select v-model="speed" :items="speeds" :label="$t('tts.speed')" hide-details></v-select>
          </v-col>
        </v-row>

        <v-alert v-if="error" type="error" class="mt-3" dense>{{ error }}</v-alert>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text :disabled="loading" @click="ttsModal = false">
          {{ $t('tts.close') }}
        </v-btn>
        <v-btn
          color="primary"
          depressed
          :loading="loading"
          :disabled="text.length === 0"
          @click="playNow"
        >
          <v-icon left>mdi-play</v-icon>
          {{ $t('tts.play') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import Vue from 'vue';

export default Vue.extend({
  name: 'TtsModal',
  data() {
    return {
      ttsModal: false,
      text: '',
      language: 'id',
      speed: 100,
      loading: false,
      error: '',
      languages: [
        { name: 'Indonesia', code: 'id' },
        { name: 'Jawa', code: 'jv' },
        { name: 'Sunda', code: 'su' },
        { name: 'English (US)', code: 'en' },
      ],
      speeds: [50, 75, 100, 125, 150],
    };
  },
  methods: {
    async playNow(): Promise<void> {
      if (!this.text.trim()) {
        return;
      }
      this.loading = true;
      this.error = '';
      try {
        const playing = await window.playTTS(this.text.trim(), this.language, this.speed);
        if (playing) {
          this.$store.commit('addToCurrentlyPlaying', playing);
          this.ttsModal = false;
        } else {
          this.error = this.$t('tts.failed') as string;
        }
      } catch (e) {
        console.error(e);
        this.error = this.$t('tts.failed') as string;
      }
      this.loading = false;
    },
  },
});
</script>
