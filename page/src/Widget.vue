<template>
  <div>
    <div v-if="loading" class="d-flex justify-center pa-4">
      <v-progress-circular indeterminate size="24" width="2" />
    </div>
    <div v-else-if="Object.keys(dvbDevices).length === 0" class="text-center pa-3">
      <v-icon size="32" color="grey" class="mb-2">mdi-television-classic</v-icon>
      <div class="text-body-2 text-medium-emphasis">{{ $t('plugin_dvb_drivers.no_adapters') }}</div>
    </div>
    <div v-else>
      <div v-for="(device, deviceName, index) in dvbDevices" :key="deviceName" :class="{ 'mt-2': index > 0 }">
        <div class="d-flex align-center mb-1" style="min-width: 0">
          <v-icon size="16" class="mr-2" style="flex-shrink: 0">mdi-television-classic</v-icon>
          <span class="text-body-2 font-weight-bold" style="white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-width: 0; flex: 1">
            {{ $t('plugin_dvb_drivers.adapter') }} {{ index + 1 }}
          </span>
        </div>
        <div class="d-flex align-center mb-1" style="gap: 6px">
          <span class="text-caption" style="width: 80px; flex-shrink: 0"><b>{{ $t('plugin_dvb_drivers.path') }}</b></span>
          <span class="text-caption font-weight-medium">/dev/{{ deviceName }}</span>
        </div>
        <div class="d-flex align-center mb-1" style="gap: 6px">
          <span class="text-caption" style="width: 80px; flex-shrink: 0"><b>{{ $t('plugin_dvb_drivers.module') }}</b></span>
          <span v-if="device.module && device.module !== 'unknown'" class="text-caption">
            {{ device.module }}
            <span v-if="device.version" class="text-medium-emphasis">v{{ device.version }}</span>
          </span>
          <span v-else class="text-caption text-medium-emphasis">{{ $t('plugin_dvb_drivers.unknown') }}</span>
        </div>
        <v-divider v-if="index < Object.keys(dvbDevices).length - 1" class="mt-2" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const loading = ref(true);
const dvbDevices = ref({});

const getAuthHeaders = () => ({
  Authorization: 'Bearer ' + localStorage.getItem('authToken'),
});

const fetchDvbDevices = async () => {
  try {
    const res = await fetch('/api/v1/mos/plugins/query', {
      method: 'POST',
      headers: {
        ...getAuthHeaders(),
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        command: 'get_dvb_devices',
        args: [],
        timeout: 10,
        parse_json: true,
      }),
    });

    if (res.ok) {
      const data = await res.json();
      dvbDevices.value = data.output || {};
    }
  } catch (e) {
    console.error('Failed to fetch DVB devices:', e);
    dvbDevices.value = {};
  }
};

onMounted(async () => {
  await fetchDvbDevices();
  loading.value = false;
});
</script>
