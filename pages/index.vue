<template>
    <div class="video" ref="artRef"></div>
</template>
<script setup>
import Artplayer from 'artplayer';
import Hls from "hls.js";
import artplayerPluginHlsQuality from "artplayer-plugin-hls-quality"
import { ref, onMounted, onBeforeUnmount, nextTick } from 'vue';

const instance = ref(null);
const artRef = ref(null);


onMounted(() => {
    instance.value = new Artplayer({
        container: artRef.value,
        url: 'https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8',
        setting: true,
        plugins: [
            artplayerPluginHlsQuality({
                // Show quality in control
                control: true,

                // Show quality in setting
                setting: true,

                // Get the resolution text from level
                getResolution: (level) => level.height + 'P',

                // I18n
                title: 'Quality',
                auto: 'Auto',
            }),
        ],
        customType: {
            m3u8: function playM3u8(video, url, art) {
                if (Hls.isSupported()) {
                    const hls = new Hls();
                    hls.loadSource(url);
                    hls.attachMedia(video);

                    console.log("video ", video);
                    console.log("url ", url);
                    console.log("art ", art);
                    console.log("hls ", hls);

                    art.hls = hls;
                    art.once('url', () => hls.destroy());
                    art.once('destroy', () => hls.destroy());
                } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
                    video.src = url;
                } else {
                    art.notice.show = 'Unsupported playback format: m3u8';
                }
            }
        },
    });
});

onBeforeUnmount(() => {
    if (instance.value) {
        instance.value.destroy(false);
    }
});
</script>

<style lang="scss">
.video {
    aspect-ratio: 16/9;
}
</style>