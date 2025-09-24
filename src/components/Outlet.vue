<template>
  <Card class="outlet-card" :class="{ 'available': !outlet.power && outlet.power !== undefined }">
    <template #content>
      <div class="outlet-content">
        <div class="power">{{ outlet.power === undefined ? '加载中' : (outlet.power || '可用') }}</div>
        <div class="duration" v-if="outlet.power">{{ outlet.used_minutes }} min</div>
      </div>
    </template>
  </Card>
</template>

<script>
import Card from 'primevue/card';

export default {
  name: 'Outlet',
  components: {
    Card
  },
  props: {
    outlet: {
      type: Object,
      required: true
    }
  }
}
</script>

<style scoped>
.outlet-card {
  width: 120px;
  text-align: center;
}

.outlet-content .power {
  font-size: 1.5rem;
  font-weight: bold;
}

.outlet-content .duration {
  font-size: 0.9rem;
  color: #6c757d;
}

:deep(.p-card-title) {
  font-size: 0.8rem;
  font-weight: normal;
  color: #6c757d;
  margin-bottom: 0.5rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

:deep(.p-card-body) {
  padding: 1rem;
}

.available {
  background-color: #e8f5e9;
  border-color: #4caf50;
}
</style>
