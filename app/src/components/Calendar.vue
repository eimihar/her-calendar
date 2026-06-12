<script setup lang="ts">
import { ref, computed } from 'vue'

interface CalendarEvent {
  id: number
  title: string
  date: string
  color: string
}

const events = ref<CalendarEvent[]>([
  { id: 1, title: 'Team Meeting', date: '2026-06-15', color: 'bg-amber-500' },
  { id: 2, title: 'Project Deadline', date: '2026-06-20', color: 'bg-rose-500' },
  { id: 3, title: 'Lunch with Sarah', date: '2026-06-18', color: 'bg-emerald-500' },
])

const currentDate = ref(new Date(2026, 5, 12))
const showModal = ref(false)
const selectedDate = ref('')
const newEventTitle = ref('')
const selectedColor = ref('bg-amber-500')

const currentMonth = computed(() => currentDate.value.getMonth())
const currentYear = computed(() => currentDate.value.getFullYear())

const monthName = computed(() => {
  return currentDate.value.toLocaleString('default', { month: 'long', year: 'numeric' })
})

const daysInMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value + 1, 0).getDate()
})

const firstDayOfMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value, 1).getDay()
})

const calendarDays = computed(() => {
  const days: (number | null)[] = []
  for (let i = 0; i < firstDayOfMonth.value; i++) {
    days.push(null)
  }
  for (let i = 1; i <= daysInMonth.value; i++) {
    days.push(i)
  }
  return days
})

const colors = ['bg-amber-500', 'bg-rose-500', 'bg-emerald-500', 'bg-orange-500', 'bg-teal-500', 'bg-pink-500']

function prevMonth() {
  currentDate.value = new Date(currentYear.value, currentMonth.value - 1, 1)
}

function nextMonth() {
  currentDate.value = new Date(currentYear.value, currentMonth.value + 1, 1)
}

function getEventsForDay(day: number) {
  const dateStr = `2026-${String(currentMonth.value + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`
  return events.value.filter(e => e.date === dateStr)
}

function openModal(day: number) {
  const dateStr = `2026-${String(currentMonth.value + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`
  selectedDate.value = dateStr
  newEventTitle.value = ''
  selectedColor.value = 'bg-amber-500'
  showModal.value = true
}

function addEvent() {
  if (!newEventTitle.value.trim()) return
  events.value.push({
    id: Date.now(),
    title: newEventTitle.value,
    date: selectedDate.value,
    color: selectedColor.value
  })
  showModal.value = false
}

function deleteEvent(id: number) {
  events.value = events.value.filter(e => e.id !== id)
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-[#292524] via-[#44403c] to-[#1c1917] p-6 md:p-8">
    <div class="max-w-4xl mx-auto">
      <div class="bg-white rounded-2xl shadow-lg border border-amber-100 overflow-hidden">
        <div class="flex items-center justify-between px-6 py-5 bg-gradient-to-r from-[#fef9f5] to-[#fffbf7] border-b border-amber-100">
          <button
            @click="prevMonth"
            class="p-2 rounded-lg hover:bg-amber-400/30 transition-all text-amber-400"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
            </svg>
          </button>
          <h2 class="text-xl md:text-2xl font-semibold text-gray-800">{{ monthName }}</h2>
          <button
            @click="nextMonth"
            class="p-2 rounded-lg hover:bg-amber-400/30 transition-all text-amber-400"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
            </svg>
          </button>
        </div>

        <div class="grid grid-cols-7">
          <div v-for="day in ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']" :key="day" class="py-3 text-center bg-[#fef9f5] border-b border-gray-100">
            <span class="text-xs font-semibold uppercase tracking-wide text-gray-500">{{ day }}</span>
          </div>

          <div
            v-for="(day, index) in calendarDays"
            :key="index"
            class="min-h-24 md:min-h-28 p-2 border-b border-r border-gray-100 bg-white cursor-pointer transition-all hover:bg-amber-50/80"
            :class="index % 7 === 6 ? 'border-r-0' : ''"
            @click="day && openModal(day)"
          >
            <template v-if="day">
              <span class="inline-flex items-center justify-center w-7 h-7 rounded-full text-sm font-medium text-gray-700 bg-gray-50">
                {{ day }}
              </span>
              <div class="mt-1.5 space-y-1">
                <div
                  v-for="event in getEventsForDay(day)"
                  :key="event.id"
                  class="text-xs px-2 py-1 rounded-md text-white truncate shadow-sm"
                  :class="event.color"
                  @click.stop
                >
                  <div class="flex items-center justify-between gap-1">
                    <span class="truncate font-medium">{{ event.title }}</span>
                    <button @click.stop="deleteEvent(event.id)" class="text-white/80 hover:text-white font-bold ml-1">×</button>
                  </div>
                </div>
              </div>
            </template>
          </div>
        </div>
      </div>
    </div>

    <Teleport to="body">
      <div v-if="showModal" class="fixed inset-0 bg-gray-900/20 backdrop-blur-sm flex items-center justify-center z-50" @click.self="showModal = false">
        <div class="bg-white rounded-2xl p-6 w-96 shadow-xl border border-gray-200">
          <h3 class="text-lg font-semibold text-gray-800 mb-1">Add Event</h3>
          <p class="text-sm text-gray-500 mb-5">{{ selectedDate }}</p>
          <input
            v-model="newEventTitle"
            type="text"
            placeholder="Event title"
            class="w-full px-4 py-3 rounded-xl bg-gray-50 border border-gray-200 text-gray-800 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-amber-400 mb-5"
            @keyup.enter="addEvent"
          />
          <div class="flex gap-3 mb-5">
            <button
              v-for="color in colors"
              :key="color"
              class="w-9 h-9 rounded-full transition-all shadow-sm"
              :class="[color, selectedColor === color ? 'ring-2 ring-offset-2 ring-gray-800 scale-110' : 'opacity-70 hover:opacity-100']"
              @click="selectedColor = color"
            />
          </div>
          <div class="flex gap-3">
            <button
              @click="showModal = false"
              class="flex-1 px-4 py-3 rounded-xl bg-gray-100 text-gray-600 hover:bg-gray-200 transition-colors font-medium"
            >
              Cancel
            </button>
            <button
              @click="addEvent"
              class="flex-1 px-4 py-3 rounded-xl bg-amber-500 text-white font-semibold hover:bg-amber-600 transition-colors shadow-sm"
            >
              Add Event
            </button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>