<script setup>
import moment from 'moment';
import { ref, onMounted } from 'vue'
import { createClient } from 'contentful'

const client = createClient({
  space: process.env.VUE_APP_SPACEID,
  accessToken: process.env.VUE_APP_TOKEN
})

const entries = ref([])
const nearestEvent = ref(null)

const formatDate = (value) => {
  if(value) {
    return moment(String(value)).format("dddd, MMMM Do YYYY, HH:mm")
  }
}

onMounted(async () => {
  try {
    const response = await client.getEntries({
      content_type: 'event'
    })
    entries.value = response.items
    findNearestEvent()
  } catch (error) {
    console.error(error)
  }
})

/* function findNearestEvent() {
  const sortedEntries = entries.value.sort((a, b) => {
    const eventDateA = new Date(a.fields.eventDate)
    const eventDateB = new Date(b.fields.eventDate)
    return eventDateA - eventDateB
  })
  nearestEvent.value = sortedEntries
} */

function findNearestEvent() {
  const currentDate = new Date()
  const upcomingEvents = entries.value.filter(entry => {
    const eventDate = new Date(entry.fields.eventDate)
    return eventDate >= currentDate
  })

  if (upcomingEvents.length > 0) {
    upcomingEvents.sort((a, b) => {
      const eventDateA = new Date(a.fields.eventDate)
      const eventDateB = new Date(b.fields.eventDate)
      return eventDateA - eventDateB
    })
    nearestEvent.value = upcomingEvents[0]
  } else {
    nearestEvent.value = null
  }
}

function setEventOpacity(eventDate) {
  const currentDate = new Date()
  const eventDateObj = new Date(eventDate)
  if (currentDate > eventDateObj) {
    return "none"
    // return "none"
  } // return display values and change :style to display instead of opacity if we want the event to disappear upon old date
  return "flex"
  // return "flex"
}
</script>

<template>
  <div class="app">
    <div class="geckoboard">
      <h1 class="heading">GeckoBoard 📊</h1>
      <iframe src="https://share.geckoboard.com/dashboards/LI62EHM4B5QP6EEM" frameborder="0"></iframe>
    </div>
    <div class="events">
      <h1 class="heading">Events 🎉</h1>
      <p class="underline-h">Dette sker snart</p>
      <div class="upcoming-event" v-if="nearestEvent && nearestEvent.fields">
        <img :src="nearestEvent.fields.eventImage.fields.file.url" alt="Event Image">
        <div class="text">
          <h2>{{ nearestEvent.fields.eventTitle }}</h2>
          <p>{{ formatDate(nearestEvent.fields.eventDate) }}</p>
        </div>
      </div>
      <div v-else>
        <p>No upcoming events found.</p>
      </div>
      <p class="underline-h">Kommende events</p>
      <div class="event-flex">
        <div class="item" v-for="entry in entries" :key="entry.sys.id" :style="{ display: setEventOpacity(entry.fields.eventDate) }">
          <div class="img-container">
            <img :src="entry.fields.eventImage.fields.file.url" alt="Event Image">
          </div>
          <div class="text">
            <h2>{{ entry.fields.eventTitle }}</h2>
            <p>{{ formatDate(entry.fields.eventDate) }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="sass" scoped>
@import @/assets/main.sass

.app
  font-family: 'Inter', sans-serif
  background: $white
  padding: 2rem
  width: 100%
  height: 100%
  min-height: 100vh
  box-sizing: border-box
  display: flex
  flex-wrap: wrap
  gap: 2rem

  .geckoboard
    flex: 60%

    iframe
      border-radius: 18px
      height: 520px
      width: 100%

  .events
    flex: 35%

  .heading
    font-size: clamp(24px, 8vw, 62px)
    font-weight: 700
    margin-bottom: 4rem

  .underline-h
    font-weight: 700
    text-decoration: underline
    margin-bottom: 1rem

  .upcoming-event
    display: flex
    gap: 1rem
    padding: 1rem
    align-items: center
    border-radius: 18px
    background: $purple
    margin-bottom: 2rem

    img
      width: 120px
      height: 120px
      border-radius: 8px

    .text
      color: $white
      display: flex
      flex-direction: column
      gap: 0.75rem

      h2
        font-weight: 500
        font-size: 28px

      p
        font-size: 18px
        font-weight: 300

  .event-flex
    display: flex
    flex-wrap: wrap
    gap: 2rem

    .item
      display: flex
      align-items: center
      background: $purple
      color: $white
      border-radius: 18px
      padding: 1rem
      gap: 1rem
      max-width: 200px
      opacity: 0.75

      .text
        h2
          font-size: 16px
          margin: 0 0 1rem 0
          font-weight: 500
          line-height: 1.2

        p
          font-size: 12px
          font-weight: 300

      .img-container
        width: 80px
        height: 80px

        img
          object-fit: cover
          background: black
          width: 100%
          height: 100%
          border-radius: 8px
</style>
