<template>
  <ClientOnly>
    <div
      v-if="width > 1024"
      class="group text-[12px] italic flex items-center gap-1"
    >
      <UIcon
        class="transform -rotate-12 duration-300 group-hover:animate-wave"
        name="i-ph-hand-pointing-duotone"
      />
      <p>{{ t('quote') }}</p>
    </div>
  </ClientOnly>
</template>

<script lang="ts" setup>
const { width } = useWindowSize()
const { t } = useI18n({
  useScope: 'local'
})
</script>

<i18n lang="json">
{
  "en": {
    "quote": "Hover the bold texts to find out more about me."
  },
  "fr": {
    "quote": "Survolez les textes en gras pour en savoir plus sur moi."
  }
}
</i18n>
