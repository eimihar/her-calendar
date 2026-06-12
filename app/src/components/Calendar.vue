<script setup lang="ts">
import { ref, computed, watch } from 'vue'

interface ScheduleConfig {
  parent1Name: string
  parent2Name: string
  parent1Color: string
  parent2Color: string
  pattern: 'biweekly' | 'weekly'
  startDate: string
  startWeekWith: 'parent1' | 'parent2'
}

const events = ref<{ id: number; title: string; date: string; color: string }>([])
const showScheduleModal = ref(false)
const showAddEventModal = ref(false)
const selectedDate = ref('')
const newEventTitle = ref('')
const selectedColor = ref('bg-amber-500')

const currentDate = ref(new Date(2026, 5, 12))

const scheduleConfig = ref<ScheduleConfig>({
  parent1Name: 'Me',
  parent2Name: 'Them',
  parent1Color: 'bg-amber-500',
  parent2Color: 'bg-teal-500',
  pattern: 'biweekly',
  startDate: '2026-06-01',
  startWeekWith: 'parent1'
})

const colors = ['bg-amber-500', 'bg-rose-500', 'bg-emerald-500', 'bg-orange-500', 'bg-teal-500', 'bg-pink-500']

const currentMonth = computed(() => currentDate.value.getMonth())
const currentYear = computed(() => currentDate.value.getFullYear())

const monthName = computed(() => {
  return currentDate.value.toLocaleString('default', { month: 'long', year: 'numeric' })
})

const daysInMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value + 1, 0).getDate()
})

const firstDayOfMonth = computed(() => {
  return (new Date(currentYear.value, currentMonth.value, 1).getDay() + 6) % 7
})

const calendarDays = computed(() => {
  const days: { day: number; month: number; year: number; isCurrentMonth: boolean }[] = []
  const firstDay = new Date(currentYear.value, currentMonth.value, 1)
  const startDayOfWeek = (firstDay.getDay() + 6) % 7
  
  for (let i = startDayOfWeek - 1; i >= 0; i--) {
    const date = new Date(currentYear.value, currentMonth.value, -i)
    days.push({ day: date.getDate(), month: date.getMonth(), year: date.getFullYear(), isCurrentMonth: false })
  }
  
  for (let i = 1; i <= daysInMonth.value; i++) {
    days.push({ day: i, month: currentMonth.value, year: currentYear.value, isCurrentMonth: true })
  }
  
  const totalCells = startDayOfWeek + daysInMonth.value
  const gridSize = Math.ceil(totalCells / 7) * 7
  const remaining = gridSize - days.length
  
  for (let i = 1; i <= remaining; i++) {
    const date = new Date(currentYear.value, currentMonth.value + 1, i)
    days.push({ day: date.getDate(), month: date.getMonth(), year: date.getFullYear(), isCurrentMonth: false })
  }
  
  return days
})

function getCustodyForDay(dayInfo: { day: number; month: number; year: number }): string | null {
  const dateStr = `${dayInfo.year}-${String(dayInfo.month + 1).padStart(2, '0')}-${String(dayInfo.day).padStart(2, '0')}`
  const startDate = new Date(scheduleConfig.value.startDate)
  const currentDayDate = new Date(dateStr)
  
  const diffTime = currentDayDate.getTime() - startDate.getTime()
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24))
  
  if (diffDays < 0) return null
  
  const weekDuration = scheduleConfig.value.pattern === 'biweekly' ? 14 : 7
  const weekIndex = Math.floor(diffDays / weekDuration)
  const dayInWeek = diffDays % weekDuration
  
  const isParent1Week = scheduleConfig.value.startWeekWith === 'parent1' 
    ? weekIndex % 2 === 0 
    : weekIndex % 2 === 1
  
  if (dayInWeek < 7) {
    return isParent1Week ? scheduleConfig.value.parent1Name : scheduleConfig.value.parent2Name
  } else {
    return isParent1Week ? scheduleConfig.value.parent2Name : scheduleConfig.value.parent1Name
  }
}

function getCustodyColor(dayInfo: { day: number; month: number; year: number }): string | null {
  const parent = getCustodyForDay(dayInfo)
  if (!parent) return null
  return parent === scheduleConfig.value.parent1Name 
    ? scheduleConfig.value.parent1Color 
    : scheduleConfig.value.parent2Color
}

function getEventsForDay(dayInfo: { day: number; month: number; year: number }) {
  const dateStr = `${dayInfo.year}-${String(dayInfo.month + 1).padStart(2, '0')}-${String(dayInfo.day).padStart(2, '0')}`
  return events.value.filter(e => e.date === dateStr)
}

function prevMonth() {
  currentDate.value = new Date(currentYear.value, currentMonth.value - 1, 1)
}

function nextMonth() {
  currentDate.value = new Date(currentYear.value, currentMonth.value + 1, 1)
}

function openAddEventModal(dayInfo: { day: number; month: number; year: number }) {
  const dateStr = `${dayInfo.year}-${String(dayInfo.month + 1).padStart(2, '0')}-${String(dayInfo.day).padStart(2, '0')}`
  selectedDate.value = dateStr
  newEventTitle.value = ''
  selectedColor.value = 'bg-amber-500'
  showAddEventModal.value = true
}

function addEvent() {
  if (!newEventTitle.value.trim()) return
  events.value.push({
    id: Date.now(),
    title: newEventTitle.value,
    date: selectedDate.value,
    color: selectedColor.value
  })
  showAddEventModal.value = false
}

function deleteEvent(id: number) {
  events.value = events.value.filter(e => e.id !== id)
}

function applySchedule() {
  showScheduleModal.value = false
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-[#292524] via-[#44403c] to-[#1c1917] p-6 md:p-8">
    <div class="max-w-4xl mx-auto">
      <div class="flex items-center justify-between mb-4">
        <h1 class="text-2xl font-bold text-white">Custody Calendar</h1>
        <button
          @click="showScheduleModal = true"
          class="px-4 py-2 rounded-xl bg-amber-500 text-white font-medium hover:bg-amber-600 transition-colors shadow-lg"
        >
          Configure Schedule
        </button>
      </div>

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

        <div class="flex items-center gap-4 px-6 py-3 bg-stone-50 border-b border-stone-200">
          <div class="flex items-center gap-2">
            <div class="w-3 h-3 rounded-full" :class="scheduleConfig.parent1Color"></div>
            <span class="text-sm text-gray-600">{{ scheduleConfig.parent1Name }}</span>
          </div>
          <div class="flex items-center gap-2">
            <div class="w-3 h-3 rounded-full" :class="scheduleConfig.parent2Color"></div>
            <span class="text-sm text-gray-600">{{ scheduleConfig.parent2Name }}</span>
          </div>
          <span class="text-xs text-gray-400 ml-auto">
            {{ scheduleConfig.pattern === 'biweekly' ? 'Bi-weekly' : 'Weekly' }} pattern
          </span>
        </div>

        <div class="grid grid-cols-7">
          <div v-for="day in ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']" :key="day" class="py-3 text-center bg-[#fef9f5] border-b border-gray-100">
            <span class="text-xs font-semibold uppercase tracking-wide text-gray-500">{{ day }}</span>
          </div>

          <div
            v-for="(dayInfo, index) in calendarDays"
            :key="index"
            class="min-h-24 md:min-h-28 p-2 border-b border-r border-gray-100 cursor-pointer transition-all hover:bg-amber-50/80"
            :class="[index % 7 === 6 ? 'border-r-0' : '', !dayInfo.isCurrentMonth ? 'bg-gray-100' : 'bg-white']"
            @click="openAddEventModal(dayInfo)"
          >
            <div class="flex items-center justify-between">
              <span 
                class="inline-flex items-center justify-center w-7 h-7 rounded-full text-sm font-medium"
                :class="dayInfo.isCurrentMonth ? 'text-gray-700 bg-gray-50' : 'text-gray-400 bg-gray-200/50'"
              >
                {{ dayInfo.day }}
              </span>
              <div
                v-if="getCustodyForDay(dayInfo)"
                class="w-2 h-2 rounded-full"
                :class="getCustodyColor(dayInfo)"
              ></div>
            </div>
            
            <div
              v-if="getCustodyForDay(dayInfo)"
              class="mt-1 text-xs px-2 py-1 rounded-md text-white"
              :class="getCustodyColor(dayInfo)"
            >
              {{ getCustodyForDay(dayInfo) }}
            </div>

            <div class="mt-1 space-y-1">
              <div
                v-for="event in getEventsForDay(dayInfo)"
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
          </div>
        </div>
      </div>
    </div>

    <Teleport to="body">
      <div v-if="showScheduleModal" class="fixed inset-0 bg-gray-900/40 backdrop-blur-sm flex items-center justify-center z-50" @click.self="showScheduleModal = false">
        <div class="bg-white rounded-2xl p-6 w-[480px] shadow-2xl border border-gray-200">
          <h3 class="text-lg font-semibold text-gray-800 mb-4">Configure Custody Schedule</h3>
          
          <div class="space-y-4">
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label class="block text-sm font-medium text-gray-600 mb-1">{{ scheduleConfig.parent1Name }}</label>
                <input
                  v-model="scheduleConfig.parent1Name"
                  type="text"
                  placeholder="Parent 1 name"
                  class="w-full px-3 py-2 rounded-lg bg-gray-50 border border-gray-200 text-gray-800 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-amber-400"
                />
                <div class="flex gap-2 mt-2">
                  <button
                    v-for="color in colors"
                    :key="color"
                    class="w-7 h-7 rounded-full transition-all"
                    :class="[color, scheduleConfig.parent1Color === color ? 'ring-2 ring-offset-2 ring-gray-800 scale-110' : 'opacity-60 hover:opacity-100']"
                    @click="scheduleConfig.parent1Color = color"
                  />
                </div>
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-600 mb-1">{{ scheduleConfig.parent2Name }}</label>
                <input
                  v-model="scheduleConfig.parent2Name"
                  type="text"
                  placeholder="Parent 2 name"
                  class="w-full px-3 py-2 rounded-lg bg-gray-50 border border-gray-200 text-gray-800 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-amber-400"
                />
                <div class="flex gap-2 mt-2">
                  <button
                    v-for="color in colors"
                    :key="color"
                    class="w-7 h-7 rounded-full transition-all"
                    :class="[color, scheduleConfig.parent2Color === color ? 'ring-2 ring-offset-2 ring-gray-800 scale-110' : 'opacity-60 hover:opacity-100']"
                    @click="scheduleConfig.parent2Color = color"
                  />
                </div>
              </div>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-600 mb-2">Pattern</label>
              <div class="flex gap-3">
                <button
                  @click="scheduleConfig.pattern = 'biweekly'"
                  class="flex-1 px-4 py-3 rounded-xl font-medium transition-all"
                  :class="scheduleConfig.pattern === 'biweekly' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                >
                  Bi-weekly
                </button>
                <button
                  @click="scheduleConfig.pattern = 'weekly'"
                  class="flex-1 px-4 py-3 rounded-xl font-medium transition-all"
                  :class="scheduleConfig.pattern === 'weekly' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                >
                  Weekly
                </button>
              </div>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-600 mb-2">Starting Week</label>
              <input
                v-model="scheduleConfig.startDate"
                type="date"
                class="w-full px-3 py-2 rounded-lg bg-gray-50 border border-gray-200 text-gray-800 focus:outline-none focus:ring-2 focus:ring-amber-400"
              />
              <p class="text-xs text-gray-400 mt-1">Select when the custody arrangement began</p>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-600 mb-2">Week Starts With</label>
              <div class="flex gap-3">
                <button
                  @click="scheduleConfig.startWeekWith = 'parent1'"
                  class="flex-1 px-4 py-3 rounded-xl font-medium transition-all flex items-center justify-center gap-2"
                  :class="scheduleConfig.startWeekWith === 'parent1' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                >
                  <div class="w-3 h-3 rounded-full" :class="scheduleConfig.parent1Color"></div>
                  {{ scheduleConfig.parent1Name }}
                </button>
                <button
                  @click="scheduleConfig.startWeekWith = 'parent2'"
                  class="flex-1 px-4 py-3 rounded-xl font-medium transition-all flex items-center justify-center gap-2"
                  :class="scheduleConfig.startWeekWith === 'parent2' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                >
                  <div class="w-3 h-3 rounded-full" :class="scheduleConfig.parent2Color"></div>
                  {{ scheduleConfig.parent2Name }}
                </button>
              </div>
            </div>
          </div>

          <div class="flex gap-3 mt-6">
            <button
              @click="showScheduleModal = false"
              class="flex-1 px-4 py-3 rounded-xl bg-gray-100 text-gray-600 hover:bg-gray-200 transition-colors font-medium"
            >
              Cancel
            </button>
            <button
              @click="applySchedule"
              class="flex-1 px-4 py-3 rounded-xl bg-amber-500 text-white font-semibold hover:bg-amber-600 transition-colors shadow-sm"
            >
              Apply Schedule
            </button>
          </div>
        </div>
      </div>

      <div v-if="showAddEventModal" class="fixed inset-0 bg-gray-900/40 backdrop-blur-sm flex items-center justify-center z-50" @click.self="showAddEventModal = false">
        <div class="bg-white rounded-2xl p-6 w-96 shadow-2xl border border-gray-200">
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
              @click="showAddEventModal = false"
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