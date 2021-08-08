<template>
  <h1>Events for Good</h1>
  <div class="events">
    <EventCard v-for="event in events" :key="event.id" :event="event" />

    <div class="pagination">
      <router-link
        :to="{ name: 'EventList', query: { page: page - 1 } }"
        rel="prev"
        v-show="page != 1"
        >&#60; Previous</router-link
      >
      <router-link
        id="page-nums"
        v-for="p in Math.ceil(totalEvents / 2)"
        :key="p"
        exact-active-class="active"
        :to="{ name: 'EventList', query: { page: p } }"
        rel="prev"
        >{{ p }}</router-link
      >
      <router-link
        :to="{ name: 'EventList', query: { page: page + 1 } }"
        rel="next"
        v-show="hasNextPage"
        >Next &#62;</router-link
      >
    </div>
  </div>
</template>

<script>
import EventCard from '@/components/EventCard.vue'
import EventService from '@/services/EventService.js'
import NProgress from 'nprogress'
export default {
  name: 'EventList',
  props: ['page'],
  components: {
    EventCard
  },
  data() {
    return {
      events: null,
      totalEvents: 0
    }
  },
  beforeRouteEnter(routeTo, routeFrom, next) {
    NProgress.start()
    return EventService.getEvents(2, parseInt(routeTo.query.page) || 1)
      .then(response => {
        next(comp => {
          comp.events = response.data
          comp.totalEvents = response.headers['x-total-count']
        })
      })
      .catch(() => {
        next({ name: 'NetworkError' })
      })
      .finally(() => {
        NProgress.done()
      })
  },
  beforeRouteUpdate(routeTo) {
    NProgress.start()
    EventService.getEvents(2, parseInt(routeTo.query.page) || 1)
      .then(response => {
        this.events = response.data
        this.totalEvents = response.headers['x-total-count']
      })
      .catch(() => {
        return { name: 'NetworkError' }
      })
      .finally(() => {
        NProgress.done()
      })
  },
  computed: {
    hasNextPage() {
      const totalPages = Math.ceil(this.totalEvents / 2)
      return this.page < totalPages
    },
    methods: {
      isActive(p) {
        console.log(p, this.page)
        return p === this.page ? 'active' : null
      }
    }
  }
}
</script>

<style scoped>
.events {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.pagination {
  display: flex;
  width: 290px;
  justify-content: space-evenly;
}
.pagination a {
  text-decoration: none;
  color: #2c3e50;
}
#page-nums.active {
  color: #42b983;
}
</style>
