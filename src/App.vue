<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue'
import L from 'leaflet'
import { 
  LayoutDashboard, 
  Map as MapIcon, 
  RotateCcw, 
  AlertTriangle, 
  CheckCircle2, 
  Clock, 
  Camera, 
  ChevronRight, 
  X, 
  Filter 
} from 'lucide-vue-next'

// --- TYPES ---
interface RoadDamage {
  id: string;
  coordinates: [number, number]; // [lat, lng] for Leaflet
  location: string;
  severity: 'High' | 'Medium' | 'Low';
  description: string;
  image: string;
  status: 'Pending' | 'In Progress' | 'Verified' | 'Fixed';
  reporter: string;
  date: string;
}

// --- SAMPLE DATA ---
const SAMPLE_DATA: RoadDamage[] = [
  {
    id: "1",
    coordinates: [-0.565, 101.442],
    location: "Jalan Raya Lubuk Jambi",
    severity: "High",
    description: "Lubang besar sedalam 15cm di tengah jalan utama, sangat berbahaya bagi pengendara motor saat hujan.",
    image: "https://images.unsplash.com/photo-1544148103-0773bf10d330?auto=format&fit=crop&q=80&w=800",
    status: "Pending",
    reporter: "Andi Saputra",
    date: "18 Mei 2026"
  },
  {
    id: "2",
    coordinates: [-0.535, 101.558],
    location: "Jalan Proklamasi Seberang Taluk",
    severity: "Medium",
    description: "Retakan memanjang di sisi bahu jalan yang mulai mengikis pondasi aspal.",
    image: "https://images.unsplash.com/photo-1515162816999-a0c47dc192f7?auto=format&fit=crop&q=80&w=800",
    status: "In Progress",
    reporter: "Budi Santoso",
    date: "17 Mei 2026"
  },
  {
    id: "3",
    coordinates: [-0.528, 101.547],
    location: "Simpang Tugu Kuantan Tengah",
    severity: "Low",
    description: "Aspal terkelupas sedikit di area persimpangan, perlu pelapisan ulang.",
    image: "https://plus.unsplash.com/premium_photo-1661962386121-65af97645f7c?auto=format&fit=crop&q=80&w=800",
    status: "Fixed",
    reporter: "Siti Aminah",
    date: "15 Mei 2026"
  },
  {
    id: "4",
    coordinates: [-0.512, 101.621],
    location: "Sentoja Raya Km 5",
    severity: "High",
    description: "Jalan amblas di bagian tengah akibat drainase yang buruk di bawah badan jalan.",
    image: "https://images.unsplash.com/photo-1508703358052-a50d4bde2882?auto=format&fit=crop&q=80&w=800",
    status: "Pending",
    reporter: "Hendra Wijaya",
    date: "19 Mei 2026"
  },
  {
    id: "5",
    coordinates: [-0.523, 101.692],
    location: "Jalan Utama Benai",
    severity: "Medium",
    description: "Gelombang jalan yang cukup parah akibat sering dilewati kendaraan muatan berat.",
    image: "https://images.unsplash.com/photo-1518112391188-df3ef06013d7?auto=format&fit=crop&q=80&w=800",
    status: "Verified",
    reporter: "Rina Kartika",
    date: "16 Mei 2026"
  }
];

// --- STATE ---
const selectedDamage = ref<RoadDamage | null>(null);
const isSidebarOpen = ref(true);
let map: L.Map | null = null;
const markers: L.Marker[] = [];

// --- METHODS ---
const initMap = () => {
  map = L.map('map-container', {
    zoomControl: false,
    attributionControl: false
  }).setView([-0.528, 101.547], 13);

  // Use OpenStreetMap - FREE & NO API KEY REQUIRED
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
  }).addTo(map);

  // Add Markers
  SAMPLE_DATA.forEach(item => {
    const color = item.severity === 'High' ? '#EF4444' : item.severity === 'Medium' ? '#F59E0B' : '#3B82F6';
    
    // Custom Icon (Simple Circle)
    const icon = L.divIcon({
      className: 'custom-marker',
      html: `<div style="background-color: ${color}; width: 14px; height: 14px; border-radius: 50%; border: 2px solid white; box-shadow: 0 2px 4px rgba(0,0,0,0.3);"></div>`,
      iconSize: [14, 14],
      iconAnchor: [7, 7]
    });

    const marker = L.marker(item.coordinates, { icon }).addTo(map!);
    marker.on('click', () => {
      selectItem(item);
    });
    markers.push(marker);
  });
};

const selectItem = (item: RoadDamage) => {
  selectedDamage.value = item;
  if (map) {
    map.flyTo(item.coordinates, 15, { duration: 1.5 });
  }
};

const getStatusIcon = (status: string) => {
  switch (status) {
    case 'Fixed': return CheckCircle2;
    case 'In Progress': return RotateCcw;
    case 'Verified': return Clock;
    default: return AlertTriangle;
  }
};

const getStatusColorClass = (status: string) => {
  switch (status) {
    case 'Fixed': return 'text-emerald-500';
    case 'In Progress': return 'text-amber-500';
    case 'Verified': return 'text-blue-500';
    default: return 'text-red-500';
  }
};

onMounted(async () => {
  await nextTick();
  initMap();
});
</script>

<template>
  <div class="flex h-screen bg-[#FDFCFB] text-neutral-900 overflow-hidden font-sans">
    
    <!-- --- SIDEBAR --- -->
    <aside 
      class="bg-white border-r border-neutral-200 transition-all duration-300 ease-in-out flex flex-col z-20"
      :class="isSidebarOpen ? 'w-[400px]' : 'w-16'"
    >
      <div class="p-4 border-b border-neutral-200 flex items-center justify-between h-16 shrink-0">
        <div v-if="isSidebarOpen" class="flex items-center gap-2 overflow-hidden">
          <div class="bg-neutral-900 p-1.5 rounded-lg shrink-0">
            <MapIcon class="w-5 h-5 text-white" />
          </div>
          <h1 class="font-bold text-xl tracking-tight italic whitespace-nowrap">ROADWATCH</h1>
        </div>
        <button 
          @click="isSidebarOpen = !isSidebarOpen"
          class="p-2 hover:bg-neutral-100 rounded-lg transition-colors shrink-0"
        >
          <LayoutDashboard class="w-5 h-5" />
        </button>
      </div>

      <div v-if="isSidebarOpen" class="flex-1 overflow-y-auto">
        <div class="p-4 bg-neutral-50 border-y border-neutral-100">
          <div class="flex items-center justify-between mb-4">
            <p class="text-xs font-bold uppercase tracking-widest text-neutral-400">Monitoring Dash</p>
            <Filter class="w-4 h-4 text-neutral-400" />
          </div>
          <div class="grid grid-cols-2 gap-3 mb-4">
            <div class="bg-white p-3 rounded-xl border border-neutral-200 shadow-sm">
              <p class="text-[10px] text-neutral-400 uppercase font-bold">Total Laporan</p>
              <p class="text-2xl font-mono font-bold">05</p>
            </div>
            <div class="bg-white p-3 rounded-xl border border-neutral-200 shadow-sm">
              <p class="text-[10px] text-neutral-400 uppercase font-bold">Perlu Perbaikan</p>
              <p class="text-2xl font-mono font-bold text-red-500">03</p>
            </div>
          </div>
        </div>

        <div class="p-2">
          <p class="px-4 py-2 text-xs font-bold uppercase tracking-widest text-neutral-400 mb-2">Laporan Terbaru - Teluk Kuantan</p>
          <div 
            v-for="item in SAMPLE_DATA" 
            :key="item.id"
            @click="selectItem(item)"
            class="p-4 rounded-xl cursor-pointer transition-all border mb-2 group"
            :class="selectedDamage?.id === item.id 
              ? 'bg-neutral-900 border-neutral-900 text-white shadow-lg' 
              : 'bg-white border-transparent hover:bg-neutral-50 hover:border-neutral-200'"
          >
            <div class="flex justify-between items-start mb-2">
              <p class="text-[10px] font-bold uppercase py-0.5 px-2 rounded-full"
                :class="selectedDamage?.id === item.id ? 'bg-white/20 text-white' : 'bg-neutral-100 text-neutral-600'"
              >
                {{ item.id }} #TK-2026
              </p>
              <span class="w-2 h-2 rounded-full"
                :class="{
                  'bg-red-500': item.severity === 'High',
                  'bg-amber-500': item.severity === 'Medium',
                  'bg-blue-500': item.severity === 'Low'
                }"
              />
            </div>
            <h3 class="font-bold text-sm leading-tight mb-1">{{ item.location }}</h3>
            <p class="text-xs line-clamp-1 opacity-70"
              :class="selectedDamage?.id === item.id ? 'text-neutral-300' : 'text-neutral-500'"
            >
              {{ item.description }}
            </p>
          </div>
        </div>
      </div>
    </aside>

    <!-- --- MAP SURFACE --- -->
    <main class="flex-1 relative">
      <div id="map-container" class="w-full h-full z-10"></div>

      <!-- --- TOP BAR (Floating) --- -->
      <div class="absolute top-4 left-4 right-4 flex justify-between items-center pointer-events-none z-20">
        <div class="bg-white/80 backdrop-blur-md px-4 py-2 rounded-full border border-white/50 shadow-lg flex items-center gap-3 pointer-events-auto">
          <span class="w-2 h-2 bg-emerald-500 rounded-full animate-pulse" />
          <p class="text-xs font-bold uppercase tracking-tight text-neutral-700">Teluk Kuantan Live Monitor (No Key Needed)</p>
          <div class="h-4 w-[1px] bg-neutral-300" />
          <p class="text-[10px] font-mono text-neutral-500">2026-05-19 17:15:00</p>
        </div>
      </div>
      
      <!-- --- DETAIL PANEL (Overlay) --- -->
      <Transition
        enter-active-class="transform transition ease-out duration-300"
        enter-from-class="translate-x-full"
        enter-to-class="translate-x-0"
        leave-active-class="transform transition ease-in duration-300"
        leave-from-class="translate-x-0"
        leave-to-class="translate-x-full"
      >
        <div 
          v-if="selectedDamage"
          class="absolute top-0 right-0 h-full w-[400px] bg-white border-l border-neutral-200 shadow-2xl z-30 flex flex-col"
        >
          <div class="p-6 border-b border-neutral-100 flex items-center justify-between h-16 shrink-0">
            <div class="overflow-hidden">
              <p class="text-[10px] font-bold text-neutral-400 uppercase tracking-[0.2em] mb-0.5">Detail Kerusakan</p>
              <h2 class="text-lg font-bold italic tracking-tight truncate">{{ selectedDamage.location }}</h2>
            </div>
            <button 
              @click="selectedDamage = null"
              class="p-2 hover:bg-neutral-100 rounded-full transition-colors shrink-0"
            >
              <X class="w-5 h-5" />
            </button>
          </div>

          <div class="flex-1 overflow-y-auto">
            <div class="aspect-video w-full relative group shrink-0">
              <img 
                :src="selectedDamage.image" 
                :alt="selectedDamage.location"
                class="w-full h-full object-cover"
              />
              <div class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent opacity-0 group-hover:opacity-100 transition-opacity flex items-end p-4">
                <div class="flex items-center gap-2 text-white">
                  <Camera class="w-4 h-4" />
                  <span class="text-xs font-medium italic">Bukti Visual - {{ selectedDamage.reporter }}</span>
                </div>
              </div>
            </div>

            <div class="p-6">
              <div class="flex items-center gap-4 mb-8">
                <div class="flex-1">
                  <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Status</p>
                  <div class="flex items-center gap-2 bg-neutral-50 p-2 rounded-lg border border-neutral-100">
                    <component :is="getStatusIcon(selectedDamage.status)" :class="['w-4 h-4', getStatusColorClass(selectedDamage.status)]" />
                    <span class="text-sm font-bold">{{ selectedDamage.status }}</span>
                  </div>
                </div>
                <div class="flex-1">
                  <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Tingkat Bahaya</p>
                  <div class="w-full h-4 rounded-full overflow-hidden flex"
                    :class="selectedDamage.severity === 'High' ? 'bg-red-100' : selectedDamage.severity === 'Medium' ? 'bg-amber-100' : 'bg-blue-100'"
                  >
                    <div class="h-full"
                      :class="selectedDamage.severity === 'High' ? 'w-full bg-red-500' : selectedDamage.severity === 'Medium' ? 'w-2/3 bg-amber-500' : 'w-1/3 bg-blue-500'"
                    />
                  </div>
                </div>
              </div>

              <div class="space-y-6">
                <section>
                  <h4 class="flex items-center gap-2 text-xs font-bold uppercase text-neutral-400 mb-3">
                    <LayoutDashboard class="w-3 h-3" />
                    Deskripsi Temuan
                  </h4>
                  <p class="text-neutral-700 leading-relaxed font-serif italic text-lg">
                    "{{ selectedDamage.description }}"
                  </p>
                </section>

                <div class="h-[1px] bg-neutral-100" />

                <section class="grid grid-cols-2 gap-4">
                  <div>
                    <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Dilaporkan Oleh</p>
                    <p class="text-sm font-bold">{{ selectedDamage.reporter }}</p>
                  </div>
                  <div>
                    <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Tanggal Laporan</p>
                    <p class="text-sm font-bold">{{ selectedDamage.date }}</p>
                  </div>
                </section>

                <section>
                  <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Koordinat Presisi</p>
                  <code class="text-[10px] flex items-center gap-1 bg-neutral-900 text-white p-2 rounded-md font-mono">
                    {{ selectedDamage.coordinates[0].toFixed(6) }}, {{ selectedDamage.coordinates[1].toFixed(6) }}
                  </code>
                </section>
              </div>
            </div>
          </div>

          <div class="p-6 border-t border-neutral-100 shrink-0">
            <button class="w-full bg-neutral-900 text-white font-bold py-4 rounded-2xl flex items-center justify-center gap-2 hover:bg-neutral-800 transition-colors shadow-lg">
              <span>TINDAK LANJUT</span>
              <ChevronRight class="w-4 h-4" />
            </button>
          </div>
        </div>
      </Transition>
    </main>
  </div>
</template>

<style>
.custom-marker {
  background: transparent;
  border: none;
}

/* Ensure leaflet container has height */
#map-container {
  background: #f8f9fa;
}

/* Override Leaflet default z-index issues with tooltips if any */
.leaflet-container {
  font-family: inherit;
}
</style>
