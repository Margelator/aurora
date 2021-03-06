<template>
  <div id="home">
    <Transition name="fade-transform" mode="out-in">
      <div class="main" v-if="posts.length">
        <article
          class="card"
          data-aos="fade-up"
          v-for="post in posts"
          :key="post.id"
          @click="gotoPost(post.number)"
          @mouseenter="showTips(post)"
        >
          <div class="post-header">
            <Cover
              :src="post.cover.src"
              :alt="post.cover.title"
              :loadCover="post.loadCover"
              :isLoad="post.isLoad"
              @loadNextCover="loadNextCover(post)"
            />
            <div class="head">
              <h3>{{ post.title }}</h3>
              <span>{{ post.cover.title }}</span>
            </div>
          </div>
          <div class="post-body"><MarkDown :content="post.description" /></div>
          <div class="post-meta">
            <span> <i class="icon icon-calendar"></i> {{ post.created_at }} </span>
            <span> <i class="icon icon-fire"></i> 热度{{ times[post.id] || 1 }}℃ </span>
            <span>
              <i class="icon icon-bookmark-empty"></i> {{ post.milestone ? post.milestone.title : '未分类' }}
            </span>
            <span>
              <i class="icon icon-tag"></i>
              <span v-for="label in post.labels.slice(0, 2)" :key="label.id">{{ label.name }}</span>
            </span>
          </div>
        </article>
      </div>
    </Transition>

    <Transition name="fade-transform" mode="out-in">
      <div v-if="!list.length"><Loading /></div>
      <div class="btn-group" v-if="list.length && (!isDisabledPrev || !isDisabledNext)">
        <Pagination
          :loading="loading"
          :isDisabledPrev="isDisabledPrev"
          :isDisabledNext="isDisabledNext"
          @handlePage="queryPosts"
        />
      </div>
    </Transition>
  </div>
</template>

<script>
import AOS from 'aos'
import { mapState } from 'vuex'
import MarkDown from '@/components/MarkDown'
import Loading from '@/components/Loading'
import Pagination from '@/components/Pagination'
import Cover from '@/components/Cover'

export default {
  name: 'Home',
  components: {
    MarkDown,
    Loading,
    Pagination,
    Cover
  },
  data() {
    return {
      loading: false,
      page: 0,
      pageSize: 12,
      posts: [],
      list: [],
      times: {}
    }
  },
  computed: {
    ...mapState({
      totalCount: state => state.totalCount
    }),
    currentCount() {
      let count = 0
      this.list.forEach((o, i) => {
        if (i <= this.page) {
          count += o.length
        }
      })
      return count
    },
    isDisabledPrev() {
      return this.page <= 1
    },
    isDisabledNext() {
      return this.currentCount >= this.totalCount
    }
  },
  async created() {
    if (!this.totalCount) {
      await this.$store.dispatch('queryArchivesCount')
    }
    await this.queryPosts()

    AOS.init({
      duration: 2000,
      easing: 'ease-out',
      debounceDelay: 200,
      offset: 50
    })
  },
  methods: {
    // 获取文章列表
    async queryPosts(type = 'next') {
      if (this.loading) return
      const queryPage = type === 'prev' ? this.page - 1 : this.page + 1

      if (this.list[queryPage]) {
        this.scrollTop(() => {
          this.page = queryPage
          this.posts = this.list[queryPage]
        })
        return
      }

      this.loading = true
      const posts = await this.$store.dispatch('queryPosts', {
        page: queryPage,
        pageSize: this.pageSize
      })
      this.loading = false

      this.scrollTop(() => {
        this.page = queryPage
        this.posts = posts
        this.$set(this.list, queryPage, posts)
      })

      // 获取文章热度
      this.$nextTick(async () => {
        const ids = posts.map(o => o.id)
        const hot = await this.$store.dispatch('queryHot', { ids })
        this.times = { ...this.times, ...hot }
      })
    },
    // 滚动到顶部
    scrollTop(cb) {
      window.scrollTo({ top: 0, behavior: 'smooth' })
      const delayTime = this.$isMobile ? 500 : 0
      setTimeout(cb, 1000 + delayTime)
      setTimeout(AOS.refresh, 1500 + delayTime)
    },
    // 按顺序加载封面图
    loadNextCover(post) {
      post.isLoad = true
      const nextPost = this.posts.find(o => !o.loadCover)
      if (nextPost) nextPost.loadCover = true
    },
    // 跳转文章页
    gotoPost(number) {
      this.$router.push({ name: 'post', params: { number } })
    },
    // 看板娘
    showTips(post) {
      const tips = `要去看看<span style="color: #b854d4"> ${post.title} </span>吗？`
      this.$store.dispatch('showTips', { tips })
    }
  }
}
</script>

<style lang="less" scope>
@import './index.less';
</style>
