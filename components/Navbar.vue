<template>
  <header class="navbar">
    <div class="inner">
      <SidebarButton @toggle-sidebar="$emit('toggle-sidebar')"/>

      <router-link
        :to="$localePath"
        class="home-link"
        v-show="!isSearching"
      >
        <img
          class="logo"
          v-if="$site.themeConfig.logo"
          :src="$withBase($site.themeConfig.logo)"
          :alt="$siteTitle"
        >
        <span
          ref="siteName"
          class="site-name"
          v-if="$siteTitle"
          :class="{ 'can-hide': $site.themeConfig.logo }"
        >{{ $siteTitle }}</span>
      </router-link>

      <div
        class="links"
        :style="linksWrapMaxWidth ? {
          'max-width': linksWrapMaxWidth + 'px'
        } : {}"
      >
        <span class="stretched-box"/>
        <AlgoliaSearchBox
          v-if="isAlgoliaSearch"
          :options="algolia"
        />
        <SearchBox ref="searchBox" v-else-if="$site.themeConfig.search !== false && $page.frontmatter.search !== false"/>
        <NavLinks class="can-hide"/>
      </div>
    </div>
  </header>
</template>

<script>
import AlgoliaSearchBox from '@AlgoliaSearchBox'
import SearchBox from '@SearchBox'
import SidebarButton from '@theme/components/SidebarButton.vue'
import NavLinks from '@theme/components/NavLinks.vue'

export default {
  components: { SidebarButton, NavLinks, SearchBox, AlgoliaSearchBox },

  data () {
    return {
      linksWrapMaxWidth: null,
      isSearching: false
    }
  },

  mounted () {
    const MOBILE_DESKTOP_BREAKPOINT = 719 // refer to config.styl
    const NAVBAR_VERTICAL_PADDING = parseInt(css(this.$el, 'paddingLeft')) + parseInt(css(this.$el, 'paddingRight'))
    const handleLinksWrapWidth = () => {
      if (document.documentElement.clientWidth < MOBILE_DESKTOP_BREAKPOINT) {
        this.linksWrapMaxWidth = null
      } else {
        this.linksWrapMaxWidth = this.$el.offsetWidth - NAVBAR_VERTICAL_PADDING
          - (this.$refs.siteName && this.$refs.siteName.offsetWidth || 0)
      }
    }
    handleLinksWrapWidth()
    window.addEventListener('resize', handleLinksWrapWidth, false)
    // hide logo when searching in mobile
    this.$refs.searchBox.$watch('focused', focused => {
      if (document.documentElement.clientWidth < MOBILE_DESKTOP_BREAKPOINT) {
        this.isSearching = focused
      }
    })
  },

  computed: {
    algolia () {
      return this.$themeLocaleConfig.algolia || this.$site.themeConfig.algolia || {}
    },

    isAlgoliaSearch () {
      return this.algolia && this.algolia.apiKey && this.algolia.indexName
    }
  }
}

function css (el, property) {
  // NOTE: Known bug, will return 'auto' if style value is 'auto'
  const win = el.ownerDocument.defaultView
  // null means not to return pseudo styles
  return win.getComputedStyle(el, null)[property]
}
</script>

<style lang="stylus">
$navbar-vertical-padding = 0.7rem
$navbar-horizontal-padding = 1.5rem

$headerTextColor = #ffffff
$headerBg = #282c34

.navbar
  padding $navbar-vertical-padding $navbar-horizontal-padding
  line-height 2.5rem
  background-color $headerBg
  .inner
    max-width 1000px
    margin 0 auto
  .home-link
    display block
    float left
  a, span, img
    display inline-block
  .logo
    height 100%
    margin-right 0.8rem
    vertical-align top
  .site-name
    font-size 1.3rem
    font-weight 600
    color $headerTextColor
    position relative
  .links
    padding-left 1.5rem
    box-sizing border-box
    background-color $headerBg
    white-space nowrap
    font-size 0.9rem
    position relative
    display flex
    color $headerTextColor
    .stretched-box
      flex 1
    .search-box
      flex: 0 0 auto
      vertical-align top
      input
        color #CCC
        border 1px solid #282c34
        background-color #1d2025

@media (max-width: $MQMobile)
  .navbar
    padding-left 4rem
    .can-hide
      display none
    .links
      padding-top 0.1rem
      padding-left 1rem
</style>
