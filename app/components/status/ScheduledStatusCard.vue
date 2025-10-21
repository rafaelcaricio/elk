<script setup lang="ts">
import type { mastodon } from 'masto'

const props = defineProps<{
  scheduledStatus: mastodon.v1.ScheduledStatus
}>()

const emit = defineEmits<{
  cancel: [id: string]
  update: [id: string, scheduledAt: string]
}>()

const { scheduledStatus: localScheduledStatus, isDeleting, isUpdating, cancelScheduledStatus, updateScheduledTime } = useScheduledStatusActions({ scheduledStatus: toRef(props, 'scheduledStatus') })

const scheduledAt = useFormattedDateTime(() => localScheduledStatus.value.scheduledAt)
const timeAgoOptions = useTimeAgoOptions(true)
const timeago = useTimeAgo(() => localScheduledStatus.value.scheduledAt, timeAgoOptions)

const showEditDialog = ref(false)
const newScheduledTime = ref('')

// Get minimum datetime (now)
const minDateTime = computed(() => {
  const now = new Date()
  const year = now.getFullYear()
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const day = String(now.getDate()).padStart(2, '0')
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  return `${year}-${month}-${day}T${hours}:${minutes}`
})

async function handleCancel() {
  const result = await cancelScheduledStatus()
  if (result)
    emit('cancel', localScheduledStatus.value.id)
}

function openEditDialog() {
  // Convert ISO string to datetime-local format
  if (localScheduledStatus.value.scheduledAt) {
    const date = new Date(localScheduledStatus.value.scheduledAt)
    const year = date.getFullYear()
    const month = String(date.getMonth() + 1).padStart(2, '0')
    const day = String(date.getDate()).padStart(2, '0')
    const hours = String(date.getHours()).padStart(2, '0')
    const minutes = String(date.getMinutes()).padStart(2, '0')
    newScheduledTime.value = `${year}-${month}-${day}T${hours}:${minutes}`
  }
  showEditDialog.value = true
}

async function handleUpdate() {
  if (!newScheduledTime.value)
    return

  // Convert datetime-local to ISO string
  const isoTime = new Date(newScheduledTime.value).toISOString()
  const result = await updateScheduledTime(isoTime)
  if (result) {
    showEditDialog.value = false
    emit('update', result.id, result.scheduledAt)
  }
}
</script>

<template>
  <article
    flex="~ col gap-2"
    border="~ base rounded-lg"
    p="4"
    bg="base"
  >
    <div flex="~ gap-2" items-center justify-between>
      <div flex="~ gap-2" items-center text-sm text-secondary>
        <div i-ri:calendar-schedule-line />
        <time :datetime="localScheduledStatus.scheduledAt" :title="scheduledAt">
          {{ timeago }}
        </time>
      </div>
      <div flex="~ gap-2">
        <button
          btn-text
          text-sm
          :disabled="isUpdating"
          @click="openEditDialog"
        >
          <div i-ri:edit-line />
          {{ $t('action.edit') }}
        </button>
        <button
          btn-text
          text-sm
          text-red
          :disabled="isDeleting"
          @click="handleCancel"
        >
          <div i-ri:delete-bin-line />
          {{ $t('action.delete') }}
        </button>
      </div>
    </div>

    <div v-if="localScheduledStatus.params.spoilerText" text-secondary>
      <strong>CW:</strong> {{ localScheduledStatus.params.spoilerText }}
    </div>

    <div>
      {{ localScheduledStatus.params.text }}
    </div>

    <div v-if="localScheduledStatus.mediaAttachments && localScheduledStatus.mediaAttachments.length > 0" flex="~ wrap gap-2">
      <div
        v-for="media in localScheduledStatus.mediaAttachments"
        :key="media.id"
        class="media-preview"
      >
        <img v-if="media.previewUrl" :src="media.previewUrl" :alt="media.description || ''" w-20 h-20 object-cover rounded>
      </div>
    </div>

    <div v-if="localScheduledStatus.params.visibility" flex="~ gap-2" items-center text-xs text-secondary>
      <div v-if="localScheduledStatus.params.visibility === 'public'" i-ri:earth-line />
      <div v-else-if="localScheduledStatus.params.visibility === 'unlisted'" i-ri:lock-unlock-line />
      <div v-else-if="localScheduledStatus.params.visibility === 'private'" i-ri:lock-line />
      <div v-else-if="localScheduledStatus.params.visibility === 'direct'" i-ri:mail-line />
      <span>{{ localScheduledStatus.params.visibility }}</span>
    </div>

    <!-- Edit dialog -->
    <div v-if="showEditDialog" fixed inset-0 z-100 flex items-center justify-center bg-black:50>
      <div bg-base border="~ base rounded-lg" p-6 max-w-md w-full m-4>
        <h3 text-lg font-bold mb-4>
          {{ $t('scheduled_status.edit_time') }}
        </h3>
        <input
          v-model="newScheduledTime"
          type="datetime-local"
          :min="minDateTime"
          w-full
          p-2
          border="~ base rounded"
          bg-base
          mb-4
        >
        <div flex="~ gap-2" justify-end>
          <button btn-text @click="showEditDialog = false">
            {{ $t('action.cancel') }}
          </button>
          <button btn-solid :disabled="!newScheduledTime || isUpdating" @click="handleUpdate">
            {{ $t('action.save') }}
          </button>
        </div>
      </div>
    </div>
  </article>
</template>
