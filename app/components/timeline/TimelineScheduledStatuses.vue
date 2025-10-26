<script setup lang="ts">
const { scheduledStatuses, isLoading, error, fetchScheduledStatuses, cancelScheduledStatus, updateScheduledStatus } = useScheduledStatuses()
const { t } = useI18n()

// Fetch scheduled statuses on mount
onMounted(() => {
  fetchScheduledStatuses()
})

// Handle cancel event from card - remove from list
async function handleCancel(id: string) {
  await cancelScheduledStatus(id)
}

// Handle update event from card - update in list
async function handleUpdate(id: string, scheduledAt: string) {
  await updateScheduledStatus(id, scheduledAt)
}
</script>

<template>
  <div>
    <!-- Loading state -->
    <div v-if="isLoading" flex="~ col gap-4" p-4>
      <div v-for="i in 3" :key="i" animate-pulse>
        <div h-32 bg="gray-200 dark:gray-800" rounded-lg />
      </div>
    </div>

    <!-- Error state -->
    <div v-else-if="error" p-4 text-center text-red>
      <div i-ri:error-warning-line text-xl mb-2 />
      <p>{{ t('state.error') }}: {{ error }}</p>
    </div>

    <!-- Empty state -->
    <div v-else-if="scheduledStatuses.length === 0" p-8 text-center text-secondary>
      <div i-ri:calendar-schedule-line text-4xl mb-4 />
      <p text-lg>
        {{ t('scheduled_status.no_scheduled_statuses') }}
      </p>
    </div>

    <!-- List of scheduled statuses -->
    <div v-else flex="~ col gap-4" p-4>
      <ScheduledStatusCard
        v-for="scheduledStatus in scheduledStatuses"
        :key="scheduledStatus.id"
        :scheduled-status="scheduledStatus"
        @cancel="handleCancel"
        @update="handleUpdate"
      />
    </div>
  </div>
</template>
