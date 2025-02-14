<script lang="ts" setup>
import type { PropType } from 'vue'

const props = defineProps({
  poster: {
    type: String,
    default: ''
  },
  src: {
    type: String,
    default: ''
  },
  sources: {
    type: Array as PropType<any[]>,
    default: () => []
  }
})

const provider = computed(() => {
  if (props.src && props.src.includes('youtube.com/watch')) {
    const match = props.src.match(/\?v=([^&]*)/)

    return {
      name: 'youtube',
      src: `https://www.youtube-nocookie.com/embed/${match[1]}?autoplay=1`,
      poster: props.poster || `https://i3.ytimg.com/vi/${match[1]}/hqdefault.jpg`
    }
  }
})

const loaded = ref(false)

if (!props.src && !props.sources.length) { throw new Error('VideoPlayer: you need to provide either `src` or `sources` props') }

const src = computed(() => props.src || props.sources?.[0]?.src || false)
</script>

<template>
  <div class="video-player" :class="{ loaded }">
    <NuxtImg v-if="provider ? provider.poster : poster" :src="(provider ? provider.poster : poster as any)" />

    <div v-if="loaded" class="loaded">
      <!-- Remote -->
      <video v-if="!provider" :poster="poster" controls autoplay>
        <source v-if="src" :src="src">
        <source v-for="source in sources" :key="source.src || source" :src="source.src || source" :type="source.type">
      </video>

      <!-- YouTube -->
      <iframe
        v-else-if="provider.name === 'youtube'"
        allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
        allowfullscreen="true"
        :autoplay="autoplay"
        :src="provider.src"
      />
    </div>

    <div v-if="!loaded" class="play-button" @click="loaded = true">
      <button />
    </div>
  </div>
</template>

<style scoped lang="ts">
css({
  '.video-player': {
    position: 'relative',
    display: 'inline-block',
    my: '{space.4}',
    overflow: 'hidden',
    background: '{colors.gray.900}',
    borderRadius: '{radii.lg}',
    '.loaded': {
      position: 'absolute',
      top: 0,
      left: 0,
      width: '100%',
      height: '100%',
    },
    video: {
      width: '100%'
    },
    iframe: {
      width: '100%',
      height: '100%',
    },
    '.play-button': {
      position: 'absolute',
      top: 0,
      left: 0,
      width: '100%',
      height: '100%',
      cursor: 'pointer',
      button: {
        position: 'absolute',
        transform: 'translate3d(-50%, -50%, 0)',
        top: '50%',
        left: '50%',
        zIndex: 1,
        width: '{space.24}',
        height: '{space.24}',
        filter: 'grayscale(100%)',
        transition: 'filter 0.1s cubic-bezier(0, 0, 0.2, 1)',
        backgroundPosition: 'center',
        backgroundRepeat: 'no-repeat',
        backgroundColor: 'transparent',
        backgroundImage: `url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 68 48"><path fill="%23f00" fill-opacity="0.8" d="M66.52,7.74c-0.78-2.93-2.49-5.41-5.42-6.19C55.79,.13,34,0,34,0S12.21,.13,6.9,1.55 C3.97,2.33,2.27,4.81,1.48,7.74C0.06,13.05,0,24,0,24s0.06,10.95,1.48,16.26c0.78,2.93,2.49,5.41,5.42,6.19 C12.21,47.87,34,48,34,48s21.79-0.13,27.1-1.55c2.93-0.78,4.64-3.26,5.42-6.19C67.94,34.95,68,24,68,24S67.94,13.05,66.52,7.74z"></path><path d="M 45,24 27,14 27,34" fill="%23fff"></path></svg>')`
      },
      '&:hover button': {
        filter: 'none'
      }
    }
  }
})
</style>