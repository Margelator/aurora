<template>
  <div id="header">
    <div class="inner">
      <RouterLink class="title" to="/">{{ $config.title }}</RouterLink>
      <span class="subtitle">{{ $config.subtitle }}</span>
      <ul>
        <li v-for="menu in displayMenus" :key="menu.path">
          <RouterLink :to="menu.path">
            <i :class="['icon', `icon-${menu.icon}`]" /> {{ menu.title }}
          </RouterLink>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Header',
  data() {
    const {
      archiveOpts,
      categoryOpts,
      tagOpts,
      inspirationOpts,
      bookOpts,
      friendOpts,
      aboutOpts
    } = this.$config
    return {
      menus: [
        { path: '/', display: true, icon: 'shop', title: '首页' },
        { path: '/archive', display: archiveOpts.display, icon: 'inbox', title: '归档' },
        { path: '/category', display: categoryOpts.display, icon: 'bookmark-empty', title: '分类' },
        { path: '/tag', display: tagOpts.display, icon: 'tag', title: '标签' },
        { path: '/book', display: bookOpts.display, icon: 'pencil', title: '书单' },
        { path: '/inspiration', display: inspirationOpts.display, icon: 'comment', title: '灵感' },
        { path: '/friend', display: friendOpts.display, icon: 'heart', title: '友链' },
        { path: '/about', display: aboutOpts.display, icon: 'universal-access', title: '关于' }
      ]
    }
  },
  computed: {
    displayMenus() {
      return this.menus.filter(o => o.display)
    }
  }
}
</script>

<style lang="less" scope>
@import './index.less';
</style>
