<template>
  <div>
    <h2 class="mb-4">DVB Drivers</h2>
    <v-skeleton-loader v-if="loading" :loading="true" type="card, card" />
    <div v-else style="margin-bottom: 80px">
      <v-card class="mb-4 pa-0">
        <v-card-title>Driver Information</v-card-title>
        <v-card-text class="pa-4">
          <div v-if="Object.keys(dvbDevices).length === 0" class="text-center text-grey py-4">
            No Adapters found
          </div>
          <v-list v-else density="compact" class="pa-0">
            <v-list-item
              v-for="(device, deviceName, index) in dvbDevices"
              :key="deviceName"
              class="px-0"
            >
              <v-list-item-title>
                <span class="font-weight-medium">Adapter {{ index + 1 }}:</span>
                <span class="text-medium-emphasis ml-2">/dev/{{ deviceName }}</span>
                <span class="mx-2">|</span>
                <span v-if="device.module && device.module !== 'unknown'" class="font-weight-medium">
                  {{ device.module }}
                  <span v-if="device.version" class="text-medium-emphasis">v{{ device.version }}</span>
                </span>
                <span v-else class="text-medium-emphasis">Unknown</span>
              </v-list-item-title>
            </v-list-item>
          </v-list>
        </v-card-text>
      </v-card>

      <v-card class="mb-4 pa-0">
        <v-card-title>Driver Selection</v-card-title>
        <v-card-text class="pa-4">
          <v-radio-group v-model="selectedDriver" hide-details class="mt-0">
            <v-radio value="digital-devices">
              <template #label>
                <span>
                  DigitalDevices
                  <v-chip
                    v-if="settings.driver === 'digital-devices'"
                    size="x-small"
                    color="success"
                    class="ml-2"
                  >
                    selected
                  </v-chip>
                </span>
              </template>
            </v-radio>
            <v-radio value="libreelec">
              <template #label>
                <span>
                  LibreELEC
                  <v-chip
                    v-if="settings.driver === 'libreelec'"
                    size="x-small"
                    color="success"
                    class="ml-2"
                  >
                    selected
                  </v-chip>
                </span>
              </template>
            </v-radio>
          </v-radio-group>
        </v-card-text>
        <v-divider />
        <v-card-actions class="pa-4">
          <v-spacer />
          <v-btn
            color="onPrimary"
            :loading="saving"
            @click="saveSettings"
          >
            Update
          </v-btn>
          <v-btn
            color="onPrimary"
            :loading="downloading"
            @click="downloadDriver"
          >
            Download
          </v-btn>
        </v-card-actions>
      </v-card>

      <v-card v-if="driverInfo && driverInfo.package" class="mb-4 pa-0">
        <v-card-title>Locally Available Driver Package</v-card-title>
        <v-card-text class="pa-4">
          <v-row dense>
            <v-col cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Plugin</strong></div>
              <div class="text-body-2">{{ driverInfo.plugin || '-' }}</div>
            </v-col>
            <v-col cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Kernel</strong></div>
              <div class="text-body-2">{{ driverInfo.kernel || '-' }}</div>
            </v-col>
            <v-col cols="12" md="6">
              <div class="text-caption text-medium-emphasis"><strong>Package</strong></div>
              <div class="text-body-2" style="word-break: break-all">{{ driverInfo.package || '-' }}</div>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

const PLUGIN_NAME = 'dvb-drivers';

const loading = ref(true);
const saving = ref(false);
const downloading = ref(false);
const dvbDevices = ref({});
const settings = ref({ driver: 'libreelec' });
const selectedDriver = ref('libreelec');
const driverInfo = ref(null);

const installedDriver = computed(() => {
  if (!driverInfo.value || !driverInfo.value.plugin) return null;
  const plugin = driverInfo.value.plugin.toLowerCase();
  if (plugin.includes('digital-devices') || plugin.includes('digitaldevices')) {
    return 'digital-devices';
  }
  if (plugin.includes('libreelec')) {
    return 'libreelec';
  }
  return null;
});

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

const fetchSettings = async () => {
  try {
    const res = await fetch(`/api/v1/mos/plugins/settings/${PLUGIN_NAME}`, {
      headers: getAuthHeaders(),
    });
    if (res.ok) {
      const data = await res.json();
      settings.value = data;
      selectedDriver.value = data.driver || 'libreelec';
    }
  } catch (e) {
    console.error('Failed to fetch settings:', e);
  }
};

const fetchDriverInfo = async () => {
  try {
    const res = await fetch(`/api/v1/mos/plugins/driver/${PLUGIN_NAME}`, {
      headers: getAuthHeaders(),
    });
    if (res.ok) {
      const data = await res.json();
      driverInfo.value = data;
    }
  } catch (e) {
    console.error('Failed to fetch driver info:', e);
    driverInfo.value = null;
  }
};

const saveSettings = async () => {
  saving.value = true;
  try {
    const res = await fetch(`/api/v1/mos/plugins/settings/${PLUGIN_NAME}`, {
      method: 'POST',
      headers: {
        ...getAuthHeaders(),
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        driver: selectedDriver.value,
      }),
    });

    if (res.ok) {
      settings.value.driver = selectedDriver.value;
    }
  } catch (e) {
    console.error('Failed to save settings:', e);
  } finally {
    saving.value = false;
  }
};

const downloadDriver = async () => {
  downloading.value = true;
  try {
    const res = await fetch('/api/v1/mos/plugins/executefunction', {
      method: 'POST',
      headers: {
        ...getAuthHeaders(),
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        plugin: PLUGIN_NAME,
        function: 'driver_download',
        name: 'Driver download',
        restart: true,
      }),
    });

    if (!res.ok) {
      console.error('Failed to execute driver download');
    }
  } catch (e) {
    console.error('Failed to download driver:', e);
  } finally {
    downloading.value = false;
  }
};

onMounted(async () => {
  try {
    await Promise.all([
      fetchDvbDevices(),
      fetchSettings(),
      fetchDriverInfo(),
    ]);
  } catch (e) {
    console.error('Failed to initialize:', e);
  } finally {
    loading.value = false;
  }
});
</script>
