<template>
  <div class="flex justify-center">
    <article class="main-content flex flex-col px-2">
      <NuxtLink
        v-if="$auth.loggedIn && $auth.user.is_admin"
        class="hover:text-blue-500 underline"
        :to="$route.path + '/edit'"
      >
        Edit
      </NuxtLink>
      <div v-if="recipe" class="md:mt-8 mt-4">
        <div class="flex flex-col">
          <div class="inline">
            <div class="inline">
              <NuxtLink
                class="text-2xs text-gray hover:text-blue-600 py-2 tracking-widest uppercase"
                :to="categoryLink"
              >
                {{ recipe.attributes.recipeCategoryName }}
              </NuxtLink>
            </div>
          </div>
          <div class="inline-flex justify-between">
            <h1 class="font-serif text-2xl">
              {{ recipe.attributes.title }}
            </h1>
            <NuxtLink no-prefetch :to="$route.path + '/print'">
              <IconPrinter
                class="hover:text-blue-400 text-blue-900 transition-colors duration-300 cursor-pointer"
              />
            </NuxtLink>
          </div>
        </div>

        <TagDisplay
          v-if="recipe.attributes.recipeTags.length"
          :tags="recipe.attributes.recipeTags"
        />

        <BaseDivider />
        <div class="inline-flex flex-row justify-between w-full">
          <div class="md:w-1/2 flex flex-col justify-between w-full">
            <div class="flex md:flex-row justify-between w-full">
              <div>
                <span
                  class="text-2xs py-2 tracking-widest text-gray-600 uppercase"
                >
                  Posted:
                  {{ dateString }}
                </span>

                <p v-if="recipe.attributes.time" class="py-2">
                  <span class="font-bold">Time:</span>
                  {{ recipe.attributes.time }}
                </p>
                <p v-if="recipe.attributes.yield" class="py-2">
                  <span class="font-bold">Yield:</span>
                  {{ recipe.attributes.yield }}
                </p>
              </div>

              <div class="px-12 py-2">
                <StarRating :value="3" />
              </div>
            </div>

            <div
              v-if="
                recipe.attributes.mainImageUrl &&
                $config.brandConfigKey === 'cookingprofessionally'
              "
              class="md:py-2 flex flex-row items-center py-6"
            >
              <span class="text-xs tracking-widest text-gray-600 uppercase">
                Share
              </span>
              <ShareButton
                network="facebook"
                class="ml-4"
                :title="recipe.attributes.title"
                :description="recipe.attributes.description"
              >
                <IconSocialFacebook
                  class="hover:text-facebook-blue h-8 text-gray-500 transition-colors duration-300 cursor-pointer"
                />
              </ShareButton>
              <ShareButton
                network="pinterest"
                class="ml-4"
                :title="recipe.attributes.title"
                :description="recipe.attributes.description"
                :media="imageURL"
              >
                <IconSocialPinterest
                  class="hover:text-pinterest-red h-8 text-gray-500 transition-colors duration-300 cursor-pointer"
                />
              </ShareButton>
            </div>
            <!-- <div><span>Recipe Image 2</span></div> -->
            <RecipeImage
              preload
              class="md:hidden h-48 mx-auto my-2"
              :height="580"
              :width="880"
              :recipe-id="recipe.id"
            />
          </div>
          <RecipeImage
            preload
            :recipe-id="recipe.id"
            :height="580"
            :width="880"
            class="md:flex h-60 xl:h-80 hidden w-1/2 my-2"
          />
        </div>

        <BaseDivider />

        <h2 class="py-4 font-serif">
          {{ recipe.attributes.description }}
        </h2>

        <BaseDivider />

        <Ad ad-name="recipes_recipes_1" class="my-8" />

        <div class="flex flex-col">
          <div class="md:flex-row flex flex-col justify-between w-full pt-4">
            <div>
              <h2 class="py-4 text-2xl font-bold">Ingredients</h2>
              <span
                v-for="(item, index) in recipe.attributes.ingredients"
                :key="`${index}-${item}`"
                class="block"
                :class="isSection(item) ? 'pt-2' : 'pb-2'"
              >
                <span v-if="isSection(item)" class="text-lg font-bold">
                  {{ removeDelimiter(item) }}
                </span>
                <span v-else class="font-serif"> • {{ item }} </span>
              </span>
            </div>

            <div
              v-if="recipe.attributes.editorNotes"
              class="md:w-1/3 flex flex-col"
            >
              <div class="md:flex flex-col items-center hidden w-full"></div>
              <p class="pt-2">
                <span class="font-bold">Editor's Notes:</span>
              </p>
              <p class="text-sm text-gray-600">
                {{ recipe.attributes.editorNotes }}
              </p>
            </div>
          </div>

          <BaseDivider />

          <Ad ad-name="recipes_recipes_2" class="my-8" />

          <h2 class="py-4 text-2xl font-bold">Directions</h2>
          <div
            v-for="(item, index) in recipe.attributes.directions"
            :key="`${index}-${item}`"
            class="block"
            :class="isSection(item) ? 'pt-2' : 'pb-2'"
          >
            <span v-if="isSection(item)" class="text-lg font-bold">
              {{ removeDelimiter(item) }}
            </span>
            <div v-else>
              <span class="font-bold">Step {{ getStepNumber(index) }}</span>
              <p class="font-serif">{{ item }}</p>
            </div>
          </div>
        </div>
        <BaseDivider />

        <Ad ad-name="recipes_recipes_3" class="my-8" />

        <RecipeFooter :recipes="footerRecipes" />
        <InfiniteScroller v-if="infiniteScroll" :handler="infiniteHandler" />
      </div>
    </article>
    <div class="lg:block hidden">
      <Ad
        ad-name="recipes_recipes_rightsticky"
        class="lg:flex top-28 sticky flex-col items-start hidden"
        :sizes="['lg', 'xl', 'xxl']"
        sidebar
      />
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import backendFunctions from '@/lib/backendFunctions'
import dateFunctions from '@/lib/dateFunctions'
import utilityFunctions from '@/lib/utilityFunctions'
import fflagFunctions from '@/mixins/fflagFunctions'

export default Vue.extend({
  mixins: [fflagFunctions],
  async asyncData({ $axios, route, error }) {
    try {
      const shouldRefreshRecipe = route.query.refresh === 'true'
      const recipe = await backendFunctions.fetchRecipe(
        $axios,
        route.params.slug,
        shouldRefreshRecipe
      )
      const footerRecipes = await backendFunctions.getRecipes($axios, {
        pageSize: 15,
        exclude: [recipe.id],
      })

      const originalExcludedRecipes = footerRecipes.map((r) => {
        return r.id
      })
      originalExcludedRecipes.push(recipe.id)

      return { recipe, footerRecipes, originalExcludedRecipes }
    } catch {
      return error({
        statusCode: 404,
        message:
          "We couldn't find that recipe. Here are some others we hope you'll enjoy.",
      })
    }
  },
  data() {
    return {
      sectionDelimiter: '***',
      page: 1,
      pageCount: 1,
    }
  },
  head() {
    return {
      title: this.recipe?.attributes?.title ? this.recipe.attributes.title : '',
      meta: [
        {
          hid: 'og:type',
          name: 'og:type',
          content: 'article',
        },
        {
          hid: 'og:title',
          name: 'og:title',
          content: this.recipe?.attributes?.title
            ? this.recipe?.attributes.title
            : '',
        },
        {
          hid: 'og:description',
          name: 'og:description',
          content: this.recipe?.attributes?.description
            ? this.recipe.attributes.description
            : '',
        },
        {
          hid: 'og:image',
          name: 'og:image',
          content: backendFunctions.getBackendSharingURL(
            this.$config,
            this.recipe?.attributes?.mainImageUrl
              ? this.recipe.attributes.mainImageUrl
              : ''
          ),
        },
        {
          hid: 'og:url',
          name: 'og:url',
          content: backendFunctions.getBackendSharingURL(
            this.$config,
            this.$route.path
          ),
        },
      ],
    }
  },
  computed: {
    infiniteScroll() {
      return this.evalFlag('cp.feature.infinite-scroll.recipe')
    },
    imageURL() {
      return backendFunctions.getImageURL(
        this.$axios,
        this.recipe.attributes.mainImageUrl
      )
    },
    dateString() {
      return dateFunctions.getDateString(
        new Date(this.recipe.attributes.publishedAt)
      )
    },
    categoryLink() {
      return utilityFunctions.getCategoryLink(
        this.recipe.attributes.recipeCategoryName
      )
    },
  },
  methods: {
    async infiniteHandler($state: any) {
      await backendFunctions
        .getRecipes(
          this.$axios,
          {
            exclude: this.originalExcludedRecipes,
            page: this.page,
          },
          true
        )
        .then((data) => {
          if (data.recipes.data) {
            this.footerRecipes.push(...data.recipes.data)
            $state.loaded()
          }
          this.pageCount = data.pages
          this.page += 1
          if (this.page > this.pageCount) {
            $state.complete()
          }
        })
    },
    getStepNumber(index: number): number {
      let step = 1
      for (let i = index - 1; i >= 0; i--) {
        if (this.isSection(this.recipe.attributes.directions[i])) {
          break
        } else {
          step++
        }
      }
      return step
    },
    isSection(item: String): Boolean {
      return (
        item.startsWith(this.sectionDelimiter) &&
        item.endsWith(this.sectionDelimiter)
      )
    },
    removeDelimiter(item: string): string {
      return item.substring(
        this.sectionDelimiter.length,
        item.length - this.sectionDelimiter.length
      )
    },
  },
})
</script>
