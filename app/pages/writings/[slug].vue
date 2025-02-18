<script lang="ts" setup>
const route = useRoute()
const { data: post } = await useAsyncData(`writing:${route.params.slug}`, () => queryContent(`/writings/${route.params.slug}`).findOne())
const {
  data: postDB,
  refresh
} = await useAsyncData(`writing:${route.params.slug}:db`, () => $fetch(`/api/posts/${route.params.slug}`, { method: 'POST' }))

const { locale, locales } = useI18n()
const currentLocale = computed(() => locales.value.filter(l => l.code === locale.value)[0])

const { t } = useI18n({
  useScope: 'local'
})

function top() {
  window.scrollTo({
    top: 0,
    left: 0,
    behavior: 'smooth'
  })
}

const { copy, copied } = useClipboard({
  source: `https://arthurdanjou.fr/writings/${route.params.slug}`,
  copiedDuring: 4000
})

useSeoMeta({
  title: post.value?.title,
  description: post.value?.description,
  author: 'Arthur Danjou'
})

function getDetails() {
  const likes = postDB.value?.likes ?? 0
  const views = postDB.value?.views ?? 0

  const like = likes > 1 ? t('likes.many') : t('likes.one')
  const view = views > 1 ? t('views.many') : t('views.one')

  return `${likes} ${like} · ${views} ${view}`
}

const likeCookie = useCookie<boolean>(`post:like:${route.params.slug}`, {
  maxAge: 7200
})

async function handleLike() {
  if (likeCookie.value) return
  await $fetch(`/api/posts/like/${route.params.slug}`, { method: 'PUT' })
  await refresh()
  likeCookie.value = true
}
</script>

<template>
  <main v-if="post && postDB">
    <div class="flex">
      <NuxtLinkLocale
        class="flex items-center gap-2 mb-8 group text-sm hover:text-black dark:hover:text-white duration-300"
        to="/writings"
      >
        <UIcon
          class="group-hover:-translate-x-1 transform duration-300"
          name="i-ph-arrow-left-duotone"
          size="20"
        />
        {{ t('back') }}
      </NuxtLinkLocale>
    </div>
    <UAlert
      v-if="locale !== 'en'"
      :description="t('alert.description')"
      :title="t('alert.title')"
      class="mb-8"
      color="red"
      icon="i-ph-warning-duotone"
      variant="outline"
    />
    <p class="border-l-2 pl-2 border-gray-300 dark:border-gray-700 rounded-sm">
      {{ getDetails() }}
    </p>
    <div class="mt-2">
      <div class="flex items-end gap-2 flex-wrap">
        <h1
          class="font-bold text-3xl text-black dark:text-white"
        >
          {{ post.title }}
        </h1>
        <p class="text-sm text-neutral-500">
          {{ useDateFormat(post.publishedAt, 'DD MMMM YYYY', { locales: currentLocale!.code ?? 'en' }).value }} ·
          {{ post.readingTime }}min long
        </p>
      </div>
      <p class="mt-4 text-base">
        {{ post.description }}
      </p>
    </div>
    <div
      v-if="post.cover"
      class="w-full rounded-md my-8"
    >
      <NuxtImg
        :src="`/writings/${post.cover}`"
        alt="Writing cover"
      />
    </div>
    <UDivider
      class="mt-8"
      icon="i-ph-pencil-line-duotone"
    />
    <article class="mt-8">
      <ContentRenderer :value="post">
        <ContentRendererMarkdown
          :value="post"
          class="!max-w-none prose dark:prose-invert"
        />
      </ContentRenderer>
      <UDivider
        class="my-16"
        icon="i-ph-hands-clapping-duotone"
      />
      <div class="space-y-8">
        <i18n-t
          keypath="thanks"
          tag="p"
        >
          <template #like>
            <strong>{{ t('like') }}</strong>
          </template>
        </i18n-t>
        <div class="flex gap-4 items-center flex-wrap">
          <UButton
            :label="postDB?.likes > 1 ? `${postDB?.likes} likes` : `${postDB?.likes} like`"
            :color="likeCookie ? 'red': 'white'"
            icon="i-ph-heart-duotone"
            size="lg"
            variant="solid"
            @click.prevent="handleLike()"
          />
          <UButton
            color="white"
            icon="i-ph-arrow-fat-lines-up-duotone"
            :label="t('top')"
            size="lg"
            variant="solid"
            @click.prevent="top()"
          />
          <UButton
            v-if="copied"
            color="green"
            icon="i-ph-check-square-duotone"
            :label="t('link.copied')"
            size="lg"
            variant="outline"
            @click.prevent="copy()"
          />
          <UButton
            v-else
            color="white"
            icon="i-ph-square-duotone"
            :label="t('link.copy')"
            size="lg"
            variant="solid"
            @click.prevent="copy()"
          />
        </div>
      </div>
    </article>
  </main>
</template>

<style>
.prose h2 a,
.prose h3 a,
.prose h4 a {
  @apply no-underline;
}
</style>

<i18n lang="json">
{
  "en": {
    "likes": {
      "one": "like",
      "many": "likes"
    },
    "views": {
      "one": "view",
      "many": "views"
    },
    "alert": {
      "title": "Translations alert!",
      "description": "Due to time constraints, all article translations will be available only in English. Thank you for your understanding."
    },
    "thanks": "Thanks for reading this post! If you liked it, please consider sharing it with your friends. {like}",
    "like": "Don't forget to leave a like!",
    "link": {
      "copied": "Link copied",
      "copy": "Copy link"
    },
    "top": "Go to top",
    "back": "Go back"
  },
  "fr": {
    "likes": {
      "one": "like",
      "many": "likes"
    },
    "views": {
      "one": "vue",
      "many": "vues"
    },
    "alert": {
      "title": "Attentions aux traductions!",
      "description": "Par soucis de temps, toutes les traductions des articles seront disponibles uniquement en anglais. Merci de votre compréhension."
    },
    "thanks": "Merci d'avoir lu cet article! Si vous l'avez aimé, n'hésitez pas à le partager avec vos amis. {like}",
    "like": "N'oubliez pas de laisser un like!",
    "link": {
      "copied": "Lien copié",
      "copy": "Copier le lien"
    },
    "top": "Remonter en haut",
    "back": "Retourner en arrière"
  }
}
</i18n>
