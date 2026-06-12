<script setup lang="ts">
import { ref, computed } from 'vue'

interface ScheduleRecord {
  id: string
  pattern: 'biweekly' | 'weekly'
  startDate: string
  endDate: string | null
  startWeekWith: 'parent1' | 'parent2'
}

interface CalendarDay {
  day: number
  month: number
  year: number
  isCurrentMonth: boolean
  isPast: boolean
}

interface CalendarEvent {
  id: number
  title: string
  date: string
  color: string
}

interface ParentConfig {
  parent1Name: string
  parent2Name: string
  parent1Color: string
  parent2Color: string
}

const events = ref<CalendarEvent[]>([])
const showDateModal = ref(false)
const selectedDate = ref('')
const dateModalTab = ref<'event' | 'schedule' | 'view'>('event')
const newEventTitle = ref('')
const selectedColor = ref('bg-amber-500')
const showParentModal = ref(false)
const showScheduleHistoryModal = ref(false)

const currentDate = ref(new Date(2026, 5, 12))

const today = new Date()
today.setHours(0, 0, 0, 0)

const parentConfig = ref<ParentConfig>({
  parent1Name: 'Me',
  parent2Name: 'Them',
  parent1Color: 'bg-amber-500',
  parent2Color: 'bg-teal-500'
})

const scheduleRecords = ref<ScheduleRecord[]>([
  {
    id: '1',
    pattern: 'weekly',
    startDate: '2026-06-01',
    endDate: null,
    startWeekWith: 'parent2'
  }
])

const colors = ['bg-amber-500', 'bg-rose-500', 'bg-emerald-500', 'bg-orange-500', 'bg-teal-500', 'bg-pink-500']

const currentMonth = computed(() => currentDate.value.getMonth())
const currentYear = computed(() => currentDate.value.getFullYear())

const monthName = computed(() => {
  return currentDate.value.toLocaleString('default', { month: 'long', year: 'numeric' })
})

const daysInMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value + 1, 0).getDate()
})

const calendarDays = computed(() => {
  const days: CalendarDay[] = []
  const firstDay = new Date(currentYear.value, currentMonth.value, 1)
  const startDayOfWeek = (firstDay.getDay() + 6) % 7
  
  for (let i = startDayOfWeek - 1; i >= 0; i--) {
    const date = new Date(currentYear.value, currentMonth.value, -i)
    date.setHours(0, 0, 0, 0)
    days.push({ 
      day: date.getDate(), 
      month: date.getMonth(), 
      year: date.getFullYear(), 
      isCurrentMonth: false,
      isPast: date < today
    })
  }
  
  for (let i = 1; i <= daysInMonth.value; i++) {
    const date = new Date(currentYear.value, currentMonth.value, i)
    date.setHours(0, 0, 0, 0)
    days.push({ 
      day: i, 
      month: currentMonth.value, 
      year: currentYear.value, 
      isCurrentMonth: true,
      isPast: date < today
    })
  }
  
  const totalCells = startDayOfWeek + daysInMonth.value
  const gridSize = Math.ceil(totalCells / 7) * 7
  const remaining = gridSize - days.length
  
  for (let i = 1; i <= remaining; i++) {
    const date = new Date(currentYear.value, currentMonth.value + 1, i)
    date.setHours(0, 0, 0, 0)
    days.push({ 
      day: date.getDate(), 
      month: date.getMonth(), 
      year: date.getFullYear(), 
      isCurrentMonth: false,
      isPast: date < today
    })
  }
  
  return days
})

const activeRecord = computed(() => {
  return scheduleRecords.value.find(r => r.endDate === null) || scheduleRecords.value[0]
})

const recordForSelectedDate = computed(() => {
  return scheduleRecords.value.find(r => {
    const start = new Date(r.startDate)
    const end = r.endDate ? new Date(r.endDate) : null
    const selected = new Date(selectedDate.value)
    return selected >= start && (!end || selected <= end)
  })
})

function getRecordStatus(record: ScheduleRecord): 'active' | 'past' | 'upcoming' {
  const start = new Date(record.startDate)

  if (record.endDate === null) {
    if (today >= start) {
      return 'active'
    }
    return 'upcoming'
  } else if (today >= start) {
    const end = new Date(record.endDate)
    if (today > end) {
      return 'past'
    }
    return 'active'
  } else {
    return 'upcoming';
  }
}

function getRecordForDate(dateStr: string): ScheduleRecord | null {
  const targetDate = new Date(dateStr)
  for (const record of scheduleRecords.value) {
    const startDate = new Date(record.startDate)
    const endDate = record.endDate ? new Date(record.endDate) : null
    
    if (targetDate >= startDate && (!endDate || targetDate <= endDate)) {
      return record
    }
  }
  return null
}

function getCustodyForDay(dayInfo: CalendarDay): string | null {
  const dateStr = `${dayInfo.year}-${String(dayInfo.month + 1).padStart(2, '0')}-${String(dayInfo.day).padStart(2, '0')}`
  const record = getRecordForDate(dateStr)
  
  if (!record) return null
  
  const startDate = new Date(record.startDate)
  const currentDayDate = new Date(dateStr)
  
  const diffTime = currentDayDate.getTime() - startDate.getTime()
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24))
  
  if (diffDays < 0) return null
  
  const weekDuration = record.pattern === 'biweekly' ? 14 : 7
  const weekIndex = Math.floor(diffDays / weekDuration)
  const dayInWeek = diffDays % weekDuration
  
  const isParent1Week = record.startWeekWith === 'parent1' 
    ? weekIndex % 2 === 0 
    : weekIndex % 2 === 1
  
  if (dayInWeek < 7) {
    return isParent1Week ? parentConfig.value.parent1Name : parentConfig.value.parent2Name
  } else {
    return isParent1Week ? parentConfig.value.parent2Name : parentConfig.value.parent1Name
  }
}

function getCustodyColor(dayInfo: CalendarDay): string | null {
  const parent = getCustodyForDay(dayInfo)
  if (!parent) return null
  return parent === parentConfig.value.parent1Name 
    ? parentConfig.value.parent1Color 
    : parentConfig.value.parent2Color
}

function getEventsForDay(dayInfo: CalendarDay) {
  const dateStr = `${dayInfo.year}-${String(dayInfo.month + 1).padStart(2, '0')}-${String(dayInfo.day).padStart(2, '0')}`
  return events.value.filter(e => e.date === dateStr)
}

function prevMonth() {
  currentDate.value = new Date(currentYear.value, currentMonth.value - 1, 1)
}

function nextMonth() {
  currentDate.value = new Date(currentYear.value, currentMonth.value + 1, 1)
}

function openDateModal(dayInfo: CalendarDay) {
  const dateStr = `${dayInfo.year}-${String(dayInfo.month + 1).padStart(2, '0')}-${String(dayInfo.day).padStart(2, '0')}`
  selectedDate.value = dateStr
  dateModalTab.value = 'event'
  newEventTitle.value = ''
  selectedColor.value = 'bg-amber-500'
  showDateModal.value = true
}

function addEvent() {
  if (!newEventTitle.value.trim()) return
  events.value.push({
    id: Date.now(),
    title: newEventTitle.value,
    date: selectedDate.value,
    color: selectedColor.value
  })
  showDateModal.value = false
}

function deleteEvent(id: number) {
  events.value = events.value.filter(e => e.id !== id)
}

function applySchedule() {
  const activeRecord = scheduleRecords.value.find(r => r.endDate === null)
  if (activeRecord) {
    const dayBefore = new Date(selectedDate.value)
    dayBefore.setDate(dayBefore.getDate() - 1)
    activeRecord.endDate = dayBefore.toISOString().split('T')[0]
  }
  
  scheduleRecords.value.push({
    id: Date.now().toString(),
    pattern: scheduleFormConfig.value.pattern,
    startDate: selectedDate.value,
    endDate: null,
    startWeekWith: scheduleFormConfig.value.startWeekWith
  })
  showDateModal.value = false
}

const scheduleFormConfig = ref({
  pattern: 'biweekly' as 'biweekly' | 'weekly',
  startWeekWith: 'parent1' as 'parent1' | 'parent2'
})

function initScheduleForm() {
  if (activeRecord.value) {
    scheduleFormConfig.value.pattern = activeRecord.value.pattern
    scheduleFormConfig.value.startWeekWith = activeRecord.value.startWeekWith
  }
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-[#292524] via-[#44403c] to-[#1c1917] p-6 md:p-8">
    <div class="max-w-4xl mx-auto">
      <div class="flex items-center justify-between mb-4">
        <h1 class="text-2xl font-bold text-white">Custody Calendar</h1>
        <div class="flex items-center gap-2">
          <button
            @click="showScheduleHistoryModal = true"
            class="p-2 rounded-xl bg-gray-700 text-white hover:bg-gray-600 transition-colors shadow-lg"
            title="Schedule History"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
            </svg>
          </button>
          <button
            @click="showParentModal = true"
            class="px-4 py-2 rounded-xl bg-amber-500 text-white font-medium hover:bg-amber-600 transition-colors shadow-lg"
          >
            Parent Settings
          </button>
        </div>
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
            <div class="w-3 h-3 rounded-full" :class="parentConfig.parent1Color"></div>
            <span class="text-sm text-gray-600">{{ parentConfig.parent1Name }}</span>
          </div>
          <div class="flex items-center gap-2">
            <div class="w-3 h-3 rounded-full" :class="parentConfig.parent2Color"></div>
            <span class="text-sm text-gray-600">{{ parentConfig.parent2Name }}</span>
          </div>
          <span class="text-xs text-gray-400 ml-auto">
            {{ activeRecord?.pattern === 'biweekly' ? 'Bi-weekly' : 'Weekly' }} pattern
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
            :class="[index % 7 === 6 ? 'border-r-0' : '', !dayInfo.isCurrentMonth ? 'bg-gray-100' : dayInfo.isPast && dayInfo.isCurrentMonth ? 'bg-gray-100' : '']"
            @click="openDateModal(dayInfo)"
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
      <div v-if="showDateModal" class="fixed inset-0 bg-gray-900/40 backdrop-blur-sm flex items-center justify-center z-50" @click.self="showDateModal = false">
        <div class="bg-white rounded-2xl p-6 w-96 shadow-2xl border border-gray-200">
          <div class="flex gap-2 mb-5">
            <button
              @click="dateModalTab = 'event'; initScheduleForm()"
              class="flex-1 px-4 py-2 rounded-lg font-medium transition-all"
              :class="dateModalTab === 'event' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
            >
              Add Event
            </button>
            <button
              @click="dateModalTab = 'schedule'; initScheduleForm()"
              class="flex-1 px-4 py-2 rounded-lg font-medium transition-all"
              :class="dateModalTab === 'schedule' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
            >
              New Schedule
            </button>
            <button
              @click="dateModalTab = 'view'"
              class="flex-1 px-4 py-2 rounded-lg font-medium transition-all"
              :class="dateModalTab === 'view' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
            >
              View Schedule
            </button>
          </div>

          <template v-if="dateModalTab === 'event'">
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
                @click="showDateModal = false"
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
          </template>

          <template v-else-if="dateModalTab == 'schedule'">
            <h3 class="text-lg font-semibold text-gray-800 mb-1">New Schedule</h3>
            <p class="text-sm text-gray-500 mb-5">Starting from {{ selectedDate }}</p>
            
            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-gray-600 mb-2">Pattern</label>
                <div class="flex gap-3">
                  <button
                    @click="scheduleFormConfig.pattern = 'biweekly'"
                    class="flex-1 px-4 py-3 rounded-xl font-medium transition-all"
                    :class="scheduleFormConfig.pattern === 'biweekly' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                  >
                    Bi-weekly
                  </button>
                  <button
                    @click="scheduleFormConfig.pattern = 'weekly'"
                    class="flex-1 px-4 py-3 rounded-xl font-medium transition-all"
                    :class="scheduleFormConfig.pattern === 'weekly' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                  >
                    Weekly
                  </button>
                </div>
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-600 mb-2">Week Starts With</label>
                <div class="flex gap-3">
                  <button
                    @click="scheduleFormConfig.startWeekWith = 'parent1'"
                    class="flex-1 px-4 py-3 rounded-xl font-medium transition-all flex items-center justify-center gap-2"
                    :class="scheduleFormConfig.startWeekWith === 'parent1' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                  >
                    <div class="w-3 h-3 rounded-full" :class="parentConfig.parent1Color"></div>
                    {{ parentConfig.parent1Name }}
                  </button>
                  <button
                    @click="scheduleFormConfig.startWeekWith = 'parent2'"
                    class="flex-1 px-4 py-3 rounded-xl font-medium transition-all flex items-center justify-center gap-2"
                    :class="scheduleFormConfig.startWeekWith === 'parent2' ? 'bg-amber-500 text-white' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'"
                  >
                    <div class="w-3 h-3 rounded-full" :class="parentConfig.parent2Color"></div>
                    {{ parentConfig.parent2Name }}
                  </button>
                </div>
              </div>
            </div>

            <div class="flex gap-3 mt-6">
              <button
                @click="showDateModal = false"
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
          </template>

          <template v-else-if="dateModalTab == 'view'">
            <h3 class="text-lg font-semibold text-gray-800 mb-1">Schedule Details</h3>
            <p class="text-sm text-gray-500 mb-5">{{ selectedDate }}</p>
            
            <div class="space-y-3 max-h-48 overflow-y-auto">
              <div
                v-if="recordForSelectedDate"
                class="p-3 rounded-lg border-2 border-amber-400 bg-amber-50"
              >
                <div class="flex items-center justify-between mb-2">
                  <span class="text-xs font-medium text-amber-600 uppercase">
                    {{ recordForSelectedDate.pattern === 'biweekly' ? 'Bi-weekly' : 'Weekly' }}
                  </span>
                  <span class="text-xs px-2 py-1 rounded bg-amber-200 text-amber-700">
                    Applied
                  </span>
                </div>
                <div class="text-sm text-gray-700">
                  <span class="font-medium">From {{ recordForSelectedDate.startDate }}</span>
                  <span class="font-medium">{{ recordForSelectedDate.endDate ? ` to ${recordForSelectedDate.endDate}` : '' }}</span>
                </div>
                <div class="flex items-center gap-2 mt-2">
                  <div class="w-3 h-3 rounded-full" :class="recordForSelectedDate.startWeekWith === 'parent1' ? parentConfig.parent1Color : parentConfig.parent2Color"></div>
                  <span class="text-sm text-gray-600">
                    Starts with {{ recordForSelectedDate.startWeekWith === 'parent1' ? parentConfig.parent1Name : parentConfig.parent2Name }}
                  </span>
                </div>
              </div>
              <div v-else class="text-sm text-gray-400 text-center py-4">
                No schedule applied for this date
              </div>
            </div>

            <div class="flex gap-3 mt-6">
              <button
                @click="showDateModal = false"
                class="flex-1 px-4 py-3 rounded-xl bg-gray-100 text-gray-600 hover:bg-gray-200 transition-colors font-medium"
              >
                Close
              </button>
            </div>
          </template>
        </div>
      </div>

      <div v-if="showParentModal" class="fixed inset-0 bg-gray-900/40 backdrop-blur-sm flex items-center justify-center z-50" @click.self="showParentModal = false">
        <div class="bg-white rounded-2xl p-6 w-[480px] shadow-2xl border border-gray-200">
          <h3 class="text-lg font-semibold text-gray-800 mb-4">Parent Settings</h3>
          
          <div class="space-y-4">
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label class="block text-sm font-medium text-gray-600 mb-1">Parent 1</label>
                <input
                  v-model="parentConfig.parent1Name"
                  type="text"
                  placeholder="Parent 1 name"
                  class="w-full px-3 py-2 rounded-lg bg-gray-50 border border-gray-200 text-gray-800 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-amber-400"
                />
                <div class="flex gap-2 mt-2">
                  <button
                    v-for="color in colors"
                    :key="color"
                    class="w-7 h-7 rounded-full transition-all"
                    :class="[color, parentConfig.parent1Color === color ? 'ring-2 ring-offset-2 ring-gray-800 scale-110' : 'opacity-60 hover:opacity-100']"
                    @click="parentConfig.parent1Color = color"
                  />
                </div>
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-600 mb-1">Parent 2</label>
                <input
                  v-model="parentConfig.parent2Name"
                  type="text"
                  placeholder="Parent 2 name"
                  class="w-full px-3 py-2 rounded-lg bg-gray-50 border border-gray-200 text-gray-800 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-amber-400"
                />
                <div class="flex gap-2 mt-2">
                  <button
                    v-for="color in colors"
                    :key="color"
                    class="w-7 h-7 rounded-full transition-all"
                    :class="[color, parentConfig.parent2Color === color ? 'ring-2 ring-offset-2 ring-gray-800 scale-110' : 'opacity-60 hover:opacity-100']"
                    @click="parentConfig.parent2Color = color"
                  />
                </div>
              </div>
            </div>
          </div>

          <div class="flex gap-3 mt-6">
            <button
              @click="showParentModal = false"
              class="flex-1 px-4 py-3 rounded-xl bg-gray-100 text-gray-600 hover:bg-gray-200 transition-colors font-medium"
            >
              Close
            </button>
          </div>
        </div>
      </div>

      <div v-if="showScheduleHistoryModal" class="fixed inset-0 bg-gray-900/40 backdrop-blur-sm flex items-center justify-center z-50" @click.self="showScheduleHistoryModal = false">
        <div class="bg-white rounded-2xl p-6 w-[480px] shadow-2xl border border-gray-200 max-h-[80vh] overflow-y-auto">
          <h3 class="text-lg font-semibold text-gray-800 mb-4">Schedule History</h3>
          
          <div class="space-y-3">
            <div
              v-for="(record, index) in scheduleRecords"
              :key="record.id"
              class="p-4 rounded-lg border-2 transition-all"
              :class="{
                'border-emerald-400 bg-emerald-50': getRecordStatus(record) === 'active',
                'border-gray-300 bg-gray-50': getRecordStatus(record) === 'past',
                'border-amber-300 bg-amber-50': getRecordStatus(record) === 'upcoming'
              }"
            >
              <div class="flex items-center justify-between mb-2">
                <span class="text-xs font-medium uppercase" :class="{
                  'text-emerald-600': getRecordStatus(record) === 'active',
                  'text-gray-500': getRecordStatus(record) === 'past',
                  'text-amber-600': getRecordStatus(record) === 'upcoming'
                }">
                  {{ record.pattern === 'biweekly' ? 'Bi-weekly' : 'Weekly' }}
                </span>
                <span
                  class="text-xs px-2 py-1 rounded font-medium"
                  :class="{
                    'bg-emerald-100 text-emerald-700': getRecordStatus(record) === 'active',
                    'bg-gray-200 text-gray-500': getRecordStatus(record) === 'past',
                    'bg-amber-200 text-amber-700': getRecordStatus(record) === 'upcoming'
                  }"
                >
                  {{ getRecordStatus(record) === 'active' ? 'Active' : getRecordStatus(record) === 'past' ? 'Past' : 'Upcoming' }}
                </span>
              </div>
              <div class="text-sm text-gray-700 font-medium">
                <span>From {{ record.startDate }}</span>
<!--                <span class="text-gray-400 mx-2">→</span>-->
                <span>{{ record.endDate ? ` to ${record.endDate}` : '' }}</span>
              </div>
              <div class="flex items-center gap-2 mt-2">
                <div class="w-3 h-3 rounded-full" :class="record.startWeekWith === 'parent1' ? parentConfig.parent1Color : parentConfig.parent2Color"></div>
                <span class="text-sm text-gray-600">
                  Starts with {{ record.startWeekWith === 'parent1' ? parentConfig.parent1Name : parentConfig.parent2Name }}
                </span>
              </div>
            </div>
            <div v-if="scheduleRecords.length === 0" class="text-sm text-gray-400 text-center py-8">
              No schedule records yet
            </div>
          </div>

          <div class="flex gap-3 mt-6">
            <button
              @click="showScheduleHistoryModal = false"
              class="flex-1 px-4 py-3 rounded-xl bg-gray-100 text-gray-600 hover:bg-gray-200 transition-colors font-medium"
            >
              Close
            </button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>