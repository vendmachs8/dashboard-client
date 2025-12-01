<template>
  <div class="min-h-screen bg-[#003A70] text-white flex flex-col">
    <!-- HEADER -->
    <header class="px-5 py-6 flex items-center justify-between">
      <div class="flex items-center gap-3">
        <div
          class="w-11 h-11 rounded-full bg-white/20 flex items-center justify-center backdrop-blur-sm"
        >
          <span class="font-semibold text-sm">HS</span>
        </div>
        <p class="text-lg font-semibold">HERY SUSILO</p>
      </div>

      <div class="flex items-center gap-1.5">
        <Button
          variant="text"
          icon="pi pi-bell"
          class="text-primary-200 hover:text-primary active:text-primary"
        />
        <Button
          variant="text"
          icon="pi pi-refresh"
          class="text-primary-200 hover:text-primary active:text-primary"
        />
      </div>
    </header>

    <!-- CONTENT -->
    <main class="bg-white rounded-t-3xl p-5 text-black flex-1">
      <!-- TITLE -->
      <section>
        <div class="flex items-center justify-between mb-5">
          <Select v-model="selectedCity" :options="cities" optionLabel="name" placeholder="Select Booth" class="w-full md:w-56" />
          <Button
            variant="text"
            icon="pi pi-sliders-h"
            label="Settings"
            class="text-gray-700"
            @click="showSettings = true"
          />
        </div>

        <!-- GRID MENU (RESPONSIVE) -->
        <div class="grid grid-cols-3 sm:grid-cols-5 gap-3 mb-6">
          <!-- MENU ITEM SEMUA (TERMASUK JARINGAN) -->
          <div
            v-for="i in allMenuItems"
            :key="i.label"
            @click="setActiveMenu(i.label)"
            :class="[
              'flex flex-col items-center p-4 rounded-xl cursor-pointer transition',
              // Warna tile (button) berdasarkan status
              tileStatusClass[i.status],
              // Efek saat aktif tetap sedikit menonjol
              activeMenu === i.label ? 'scale-105 shadow' : 'hover:scale-[1.02]'
            ]"
          >
            <i
              :class="[
                'pi',
                i.icon,
                'text-2xl',
                // Ikon kembali ke default warna; tidak berubah oleh status
                activeMenu === i.label ? 'text-primary-700' : 'text-gray-500',
              ]"
            ></i>
            <span
              :class="[
                'text-xs mt-1 font-semibold',
                activeMenu === i.label ? 'text-primary-800' : 'text-gray-700',
              ]"
            >
              {{ i.label }}
            </span>
          </div>
        </div>

        <!-- CARD OMSET -->
        <div
          class="p-5 rounded-xl bg-gray-50 shadow-sm flex justify-between items-center"
        >
          <div>
            <p class="text-gray-600 text-sm">Omset</p>
            <p class="font-bold text-2xl mt-1">Rp 100.000.000</p>
          </div>
          <img
            src="/monitor.png"
            class="w-16 h-24 md:w-24 md:h-36 object-cover object-center rounded-lg"
          />
        </div>
      </section>

      <!-- DATA LOG TABLE -->
      <section class="mt-8">
        <div class="flex justify-between items-center mb-3">
          <h2 class="text-xl font-bold">Data Log</h2>
        </div>

        <DataTable
          :value="logs"
          removableSort
          stripedRows
          showGridlines
          class="rounded-xl overflow-hidden shadow text-sm"
          tableStyle="min-width: 100%"
        >
          <Column field="tanggal" header="Tanggal & Jam" sortable></Column>
          <Column field="aktivitas" header="Aktivitas" sortable></Column>

          <Column field="status" header="Status" sortable>
            <template #body="slotProps">
              <Tag
                :value="slotProps.data.status"
                :severity="statusSeverity[slotProps.data.status]"
                rounded
              />
            </template>
          </Column>
        </DataTable>
      </section>
    </main>
    <!-- SETTINGS MODAL CARD -->
    <Teleport to="body">
      <div v-if="showSettings" class="fixed inset-0 z-50">
        <div class="absolute inset-0 bg-black/40" @click="showSettings = false"></div>
        <div class="absolute left-1/2 top-24 -translate-x-1/2 w-[90%] max-w-xl">
          <div class="bg-white rounded-2xl shadow-xl overflow-hidden">
            <div class="px-5 py-4 border-b flex items-center justify-between bg-[#003a70]">
              <h3 class="text-lg font-semibold text-white">Settings</h3>
              <button class="p-2 rounded-full hover:bg-gray-100" @click="showSettings = false">
                <i class="pi pi-times text-gray-600"></i>
              </button>
            </div>
              <div class="p-5 text-gray-700">
                <div class="space-y-5">
                  <!-- Row: Suara detek wajah -->
                  <div>
                    <span class="font-medium">Suara Detect wajah</span>
                    <div class="mt-2 flex items-center gap-2">
                      <input
                        type="text"
                        :value="selectedFaceFileName"
                        placeholder="Tidak ada file"
                        readonly
                        class="flex-1 px-3 py-2 border rounded-lg text-sm bg-gray-50"
                      />
                      <input ref="faceFileInput" type="file" accept="audio/*" class="hidden" @change="onFaceFileChange" />
                      <button class="px-3 py-2 border rounded-lg bg-white hover:bg-gray-50" @click="browseFaceFile">
                        <i class="pi pi-ellipsis-h"></i>
                      </button>
                      <button
                        class="px-3 py-2 border rounded-lg bg-white hover:bg-gray-50 disabled:opacity-50"
                        :disabled="!faceAudioUrl"
                        @click="toggleFacePlayback"
                      >
                        <i :class="['pi', isFacePlaying ? 'pi-stop' : 'pi-play']"></i>
                      </button>
                    </div>
                  </div>

                  <!-- Row: Suara abaz -->
                  <div>
                    <span class="font-medium">Suara aba-aba</span>
                    <div class="mt-2 flex items-center gap-2">
                      <input
                        type="text"
                        :value="selectedAlarmFileName"
                        placeholder="Tidak ada file"
                        readonly
                        class="flex-1 px-3 py-2 border rounded-lg text-sm bg-gray-50"
                      />
                      <input ref="alarmFileInput" type="file" accept="audio/*" class="hidden" @change="onAlarmFileChange" />
                      <button class="px-3 py-2 border rounded-lg bg-white hover:bg-gray-50" @click="browseAlarmFile">
                        <i class="pi pi-ellipsis-h"></i>
                      </button>
                      <button
                        class="px-3 py-2 border rounded-lg bg-white hover:bg-gray-50 disabled:opacity-50"
                        :disabled="!alarmAudioUrl"
                        @click="toggleAlarmPlayback"
                      >
                        <i :class="['pi', isAlarmPlaying ? 'pi-stop' : 'pi-play']"></i>
                      </button>
                    </div>
                  </div>
                  <!-- Row: Suara terimakasih -->
                  <div>
                    <span class="font-medium">Suara terimakasih</span>
                    <div class="mt-2 flex items-center gap-2">
                      <input
                        type="text"
                        :value="selectedThankYouFileName"
                        placeholder="Tidak ada file"
                        readonly
                        class="flex-1 px-3 py-2 border rounded-lg text-sm bg-gray-50"
                      />
                      <input ref="thankYouFileInput" type="file" accept="audio/*" class="hidden" @change="onThankYouFileChange" />
                      <button class="px-3 py-2 border rounded-lg bg-white hover:bg-gray-50" @click="browseThankYouFile">
                        <i class="pi pi-ellipsis-h"></i>
                      </button>
                      <button
                        class="px-3 py-2 border rounded-lg bg-white hover:bg-gray-50 disabled:opacity-50"
                        :disabled="!thankYouAudioUrl"
                        @click="toggleThankYouPlayback"
                      >
                        <i :class="['pi', isThankYouPlaying ? 'pi-stop' : 'pi-play']"></i>
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            <div class="px-5 py-4 flex justify-end gap-2">
              <Button label="Batal" variant="text" class="text-gray-700" @click="showSettings = false" />
              <Button label="Simpan" class="bg-[#003a70] text-white hover:bg-primary-700" @click="saveSettings" />
            </div>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<script setup>
import { ref } from "vue";
import Select from 'primevue/select';
import FileUpload from 'primevue/fileupload';

const logs = ref([
  {
    tanggal: "2025-01-02 08:30",
    aktivitas: "Sistem menyala",
    status: "Normal",
  },
  {
    tanggal: "2025-01-02 09:15",
    aktivitas: "Login pengguna",
    status: "Sukses",
  },
  { tanggal: "2025-01-02 10:02", aktivitas: "Printer error", status: "Error" },
  {
    tanggal: "2025-01-02 11:20",
    aktivitas: "CCTV terhubung",
    status: "Normal",
  },
  {
    tanggal: "2025-01-02 12:44",
    aktivitas: "Perubahan jaringan",
    status: "Warning",
  },
]);

const statusSeverity = {
  Normal: "success",
  Sukses: "info",
  Warning: "warn",
  Error: "danger",
};

const activeMenu = ref("Jaringan"); // default menu pertama

const allMenuItems = [
  { label: "Jaringan", icon: "pi-wifi", status: "connected" },
  { label: "Printer", icon: "pi-print", status: "error" },
  { label: "Komputer", icon: "pi-desktop", status: "connected" },
  { label: "Kamera", icon: "pi-camera", status: "disconnected" },
  { label: "Lampu", icon: "pi-lightbulb", status: "connected" },
];

// Mapping warna tile (button) berdasarkan status
const tileStatusClass = {
  connected: 'bg-green-200 border border-green-200',
  disconnected: 'bg-gray-200 border border-gray-200',
  error: 'bg-red-200 border border-red-200'
};

function setActiveMenu(label) {
  activeMenu.value = label;
}

// Settings modal state
const showSettings = ref(false);
const settings = ref({
  notifications: true,
  autoRefresh: false,
});

// Audio file rows state
const selectedFaceFileName = ref('');
const selectedAlarmFileName = ref('');
const selectedThankYouFileName = ref('');
const faceAudioUrl = ref('');
const alarmAudioUrl = ref('');
const thankYouAudioUrl = ref('');
const faceFileInput = ref(null);
const alarmFileInput = ref(null);
const thankYouFileInput = ref(null);

const faceAudio = ref(null);
const alarmAudio = ref(null);
const thankYouAudio = ref(null);
const isFacePlaying = ref(false);
const isAlarmPlaying = ref(false);
const isThankYouPlaying = ref(false);

function browseFaceFile() {
  faceFileInput.value && faceFileInput.value.click();
}
function browseAlarmFile() {
  alarmFileInput.value && alarmFileInput.value.click();
}
function browseThankYouFile() {
  thankYouFileInput.value && thankYouFileInput.value.click();
}

function onFaceFileChange(e) {
  const file = e.target.files && e.target.files[0];
  stopFaceAudio();
  if (!file) {
    selectedFaceFileName.value = '';
    revokeUrl(faceAudioUrl);
    faceAudioUrl.value = '';
    return;
  }
  selectedFaceFileName.value = file.name;
  replaceUrl(faceAudioUrl, file);
}
function onAlarmFileChange(e) {
  const file = e.target.files && e.target.files[0];
  stopAlarmAudio();
  if (!file) {
    selectedAlarmFileName.value = '';
    revokeUrl(alarmAudioUrl);
    alarmAudioUrl.value = '';
    return;
  }
  selectedAlarmFileName.value = file.name;
  replaceUrl(alarmAudioUrl, file);
}
function onThankYouFileChange(e) {
  const file = e.target.files && e.target.files[0];
  stopThankYouAudio();
  if (!file) {
    selectedThankYouFileName.value = '';
    revokeUrl(thankYouAudioUrl);
    thankYouAudioUrl.value = '';
    return;
  }
  selectedThankYouFileName.value = file.name;
  replaceUrl(thankYouAudioUrl, file);
}

function revokeUrl(urlRef) {
  if (urlRef.value) URL.revokeObjectURL(urlRef.value);
}
function replaceUrl(urlRef, file) {
  revokeUrl(urlRef);
  urlRef.value = URL.createObjectURL(file);
}

function stopFaceAudio() {
  if (faceAudio.value) {
    faceAudio.value.pause();
    faceAudio.value.currentTime = 0;
    faceAudio.value = null;
  }
  isFacePlaying.value = false;
}
function stopAlarmAudio() {
  if (alarmAudio.value) {
    alarmAudio.value.pause();
    alarmAudio.value.currentTime = 0;
    alarmAudio.value = null;
  }
  isAlarmPlaying.value = false;
}
function stopThankYouAudio() {
  if (thankYouAudio.value) {
    thankYouAudio.value.pause();
    thankYouAudio.value.currentTime = 0;
    thankYouAudio.value = null;
  }
  isThankYouPlaying.value = false;
}

function toggleFacePlayback() {
  if (!faceAudioUrl.value) return;
  if (isFacePlaying.value) return stopFaceAudio();
  stopFaceAudio();
  faceAudio.value = new Audio(faceAudioUrl.value);
  faceAudio.value.onended = stopFaceAudio;
  faceAudio.value.play();
  isFacePlaying.value = true;
}
function toggleAlarmPlayback() {
  if (!alarmAudioUrl.value) return;
  if (isAlarmPlaying.value) return stopAlarmAudio();
  stopAlarmAudio();
  alarmAudio.value = new Audio(alarmAudioUrl.value);
  alarmAudio.value.onended = stopAlarmAudio;
  alarmAudio.value.play();
  isAlarmPlaying.value = true;
}
function toggleThankYouPlayback() {
  if (!thankYouAudioUrl.value) return;
  if (isThankYouPlaying.value) return stopThankYouAudio();
  stopThankYouAudio();
  thankYouAudio.value = new Audio(thankYouAudioUrl.value);
  thankYouAudio.value.onended = stopThankYouAudio;
  thankYouAudio.value.play();
  isThankYouPlaying.value = true;
}

function saveSettings() {
  // TODO: integrate with real persistence/API if needed
  showSettings.value = false;
}

const selectedCity = ref();
const cities = ref([
    { name: 'MOG', code: 'MOG' },
    { name: 'JAMBI', code: 'JAMBI' },
    { name: 'BANJARMASIN', code: 'BANJARMASIN' },
    { name: 'JATIM PARK 3', code: 'JATIM PARK 3' },
]);

</script>
