<script setup lang="ts">
import { ref, onMounted, nextTick, watch } from 'vue'
import L from 'leaflet'
import { 
  LayoutDashboard, 
  Map as MapIcon, 
  RotateCcw, 
  CheckCircle2, 
  Clock, 
  Camera, 
  ChevronRight, 
  X, 
  Filter,
  Lightbulb,
  AlertTriangle
} from 'lucide-vue-next'

// --- TYPES ---
interface PJULight {
  id: string;
  coordinates: [number, number];
  location: string;
  condition: 'Baik' | 'Rusak';
  issue?: string;
  lastMaintenance: string;
  image: string;
  status: 'Aktif' | 'Perbaikan' | 'Non-Aktif';
  reporter: string;
  type: 'LED 120W' | 'Sodium 250W';
}

// --- SAMPLE DATA (11 Points along Jalan Proklamasi) ---
const SAMPLE_DATA: PJULight[] = [
  {
    id: "PJU-001",
    coordinates: [-0.526740, 101.550089],
    location: "Jalan Proklamasi - Titik 1",
    condition: "Baik",
    lastMaintenance: "10 April 2026",
    type: "LED 120W",
    image: "https://images.unsplash.com/photo-1542332213-31f87348057f?auto=format&fit=crop&q=80&w=800",
    status: "Aktif",
    reporter: "Sistem Auto",
  },
  {
    id: "PJU-002",
    coordinates: [-0.526697, 101.550497],
    location: "Jalan Proklamasi - Titik 2",
    condition: "Rusak",
    issue: "Kabel putus akibat dahan pohon tumbang",
    lastMaintenance: "15 Maret 2026",
    type: "LED 120W",
    image: "https://images.unsplash.com/photo-1516104883495-24d1937f3bc5?auto=format&fit=crop&q=80&w=800",
    status: "Perbaikan",
    reporter: "Warga - Budi S.",
  },
  {
    id: "PJU-003",
    coordinates: [-0.526804, 101.551034],
    location: "Jalan Proklamasi - Titik 3",
    condition: "Baik",
    lastMaintenance: "02 Mei 2026",
    type: "Sodium 250W",
    image: "https://images.unsplash.com/photo-1498192055456-4071da50a58a?auto=format&fit=crop&q=80&w=800",
    status: "Aktif",
    reporter: "Sistem Auto",
  },
  {
    id: "PJU-004",
    coordinates: [-0.527061, 101.551463],
    location: "Jalan Proklamasi - Titik 4",
    condition: "Rusak",
    issue: "Lampu berkedip intensitas cahaya rendah (Balast Lemah)",
    lastMaintenance: "12 Januari 2026",
    type: "LED 120W",
    image: "https://images.unsplash.com/photo-1563823251941-b9989d1e8d97?auto=format&fit=crop&q=80&w=800",
    status: "Non-Aktif",
    reporter: "Patroli Malam",
  },
  {
    id: "PJU-005",
    coordinates: [-0.527448, 101.551827],
    location: "Jalan Proklamasi - Titik 5",
    condition: "Baik",
    lastMaintenance: "19 Mei 2026",
    type: "LED 120W",
    image: "https://images.unsplash.com/photo-1548671653-270f9ec40081?auto=format&fit=crop&q=80&w=800",
    status: "Aktif",
    reporter: "Sistem Auto",
  },
  {
    id: "PJU-006",
    coordinates: [-0.527791, 101.552063],
    location: "Jalan Proklamasi - Titik 6",
    condition: "Baik",
    lastMaintenance: "05 Mei 2026",
    type: "LED 120W",
    image: "https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&q=80&w=800",
    status: "Aktif",
    reporter: "Sistem Auto",
  },
  {
    id: "PJU-007",
    coordinates: [-0.528220, 101.552407],
    location: "Jalan Proklamasi - Titik 7",
    condition: "Rusak",
    issue: "Fitting lampu korosi",
    lastMaintenance: "20 Desember 2025",
    type: "Sodium 250W",
    image: "https://images.unsplash.com/photo-1518384401463-d3876163cca2?auto=format&fit=crop&q=80&w=800",
    status: "Perbaikan",
    reporter: "Dinas Perhubungan",
  },
  {
    id: "PJU-008",
    coordinates: [-0.528606, 101.552750],
    location: "Jalan Proklamasi - Titik 8",
    condition: "Baik",
    lastMaintenance: "18 Mei 2026",
    type: "LED 120W",
    image: "https://images.unsplash.com/photo-1520038410233-7141f77e49aa?auto=format&fit=crop&q=80&w=800",
    status: "Aktif",
    reporter: "Sistem Auto",
  },
  {
    id: "PJU-009",
    coordinates: [-0.528721, 101.553152],
    location: "Jalan Proklamasi - Titik 9",
    condition: "Rusak",
    issue: "Sensor cahaya mati (Lampu hidup 24 jam)",
    lastMaintenance: "10 Februari 2026",
    type: "LED 120W",
    image: "https://images.unsplash.com/photo-1524311588100-34994df58700?auto=format&fit=crop&q=80&w=800",
    status: "Perbaikan",
    reporter: "Warga - Andi",
  },
  {
    id: "PJU-010",
    coordinates: [-0.528719, 101.553787],
    location: "Jalan Proklamasi - Titik 10",
    condition: "Baik",
    lastMaintenance: "12 Mei 2026",
    type: "Sodium 250W",
    image: "https://images.unsplash.com/photo-1560185127-6ed189bf02f4?auto=format&fit=crop&q=80&w=800",
    status: "Aktif",
    reporter: "Sistem Auto",
  },
  {
    id: "PJU-011",
    coordinates: [-0.528821, 101.554310],
    location: "Jalan Proklamasi - Titik 11",
    condition: "Baik",
    lastMaintenance: "15 Mei 2026",
    type: "LED 120W",
    image: "https://images.unsplash.com/photo-1542332213-31f87348057f?auto=format&fit=crop&q=80&w=800",
    status: "Aktif",
    reporter: "Dinas Kebersihan",
  },
];

// --- STATE ---
const selectedPJU = ref<PJULight | null>(null);
const isSidebarOpen = ref(true);
let map: L.Map | null = null;
const markers: L.Marker[] = [];

// --- WATCHERS ---
watch(isSidebarOpen, () => {
  setTimeout(() => {
    if (map) {
      map.invalidateSize();
    }
  }, 350);
});

// --- METHODS ---
const initMap = () => {
  map = L.map('map-container', {
    zoomControl: false,
    attributionControl: false
  }).setView([-0.5277, 101.5520], 16);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
  }).addTo(map);

  SAMPLE_DATA.forEach(item => {
    const color = item.condition === 'Baik' ? '#10B981' : '#171717';
    
    const icon = L.divIcon({
      className: 'custom-marker',
      html: `
        <div style="
          background-color: white; 
          width: 32px; 
          height: 32px; 
          border-radius: 50%; 
          display: flex; 
          align-items: center; 
          justify-content: center;
          box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
          border: 2px solid ${color};
          transition: all 0.2s ease;
        ">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="${item.condition === 'Baik' ? color : 'none'}" stroke="${color}" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M15 14c.2-1 .7-1.7 1.5-2.5 1-.9 1.5-2.2 1.5-3.5A5 5 0 0 0 8 8c0 1.3.5 2.6 1.5 3.5.8.8 1.3 1.5 1.5 2.5"/>
            <path d="M9 18h6"/><path d="M10 22h4"/>
          </svg>
        </div>
      `,
      iconSize: [32, 32],
      iconAnchor: [16, 16]
    });

    const marker = L.marker(item.coordinates, { icon }).addTo(map!);
    marker.on('click', () => {
      selectItem(item);
    });
    markers.push(marker);
  });
};

const selectItem = (item: PJULight) => {
  selectedPJU.value = item;
  if (window.innerWidth < 768) {
    isSidebarOpen.value = false;
  }
  if (map) {
    map.flyTo(item.coordinates, 17, { duration: 1.5 });
  }
};

const getStatusIcon = (status: string) => {
  switch (status) {
    case 'Aktif': return CheckCircle2;
    case 'Perbaikan': return RotateCcw;
    default: return Lightbulb;
  }
};

onMounted(async () => {
  if (window.innerWidth < 768) {
    isSidebarOpen.value = false;
  }
  await nextTick();
  initMap();
});
</script>

<template>
  <div class="relative flex h-screen w-full bg-[#FDFCFB] text-neutral-900 overflow-hidden font-sans">
    
    <!-- --- BACKDROP FOR MOBILE SIDEBAR --- -->
    <div 
      v-if="isSidebarOpen"
      @click="isSidebarOpen = false"
      class="fixed inset-0 bg-neutral-950/40 backdrop-blur-[2px] z-25 md:hidden transition-opacity duration-300 pointer-events-auto"
    ></div>

    <!-- --- SIDEBAR --- -->
    <aside 
      class="bg-white transition-all duration-300 ease-in-out flex flex-col z-30 absolute md:relative h-full top-0 left-0 border-r border-neutral-200 shadow-2xl md:shadow-none"
      :class="[
        isSidebarOpen 
          ? 'translate-x-0 w-[300px] sm:w-[360px] md:w-[400px]' 
          : '-translate-x-full md:translate-x-0 md:w-16'
      ]"
    >
      <div class="p-4 border-b border-neutral-200 flex items-center justify-between h-16 shrink-0">
        <div v-if="isSidebarOpen" class="flex items-center gap-2 overflow-hidden">
          <div class="bg-amber-500 p-1.5 rounded-lg shrink-0">
            <Lightbulb class="w-5 h-5 text-white" />
          </div>
          <h1 class="font-bold text-xl tracking-tight italic whitespace-nowrap">PJU MASTER</h1>
        </div>
        <button 
          @click="isSidebarOpen = !isSidebarOpen"
          class="p-2 hover:bg-neutral-100 rounded-lg transition-colors shrink-0"
          :class="!isSidebarOpen ? 'w-full flex justify-center' : ''"
        >
          <LayoutDashboard class="w-5 h-5" />
        </button>
      </div>

      <div v-if="isSidebarOpen" class="flex-1 overflow-y-auto">
        <div class="p-4 bg-neutral-50 border-y border-neutral-100">
          <div class="flex items-center justify-between mb-4">
            <p class="text-xs font-bold uppercase tracking-widest text-neutral-400">PJU Stats</p>
            <Filter class="w-4 h-4 text-neutral-400" />
          </div>
          <div class="grid grid-cols-2 gap-3 mb-4">
            <div class="bg-white p-3 rounded-xl border border-neutral-200 shadow-sm">
              <p class="text-[10px] text-neutral-400 uppercase font-bold">Lampu Aktif</p>
              <p class="text-2xl font-mono font-bold text-emerald-500">07</p>
            </div>
            <div class="bg-white p-3 rounded-xl border border-neutral-200 shadow-sm">
              <p class="text-[10px] text-neutral-400 uppercase font-bold">Lampu Padam</p>
              <p class="text-2xl font-mono font-bold text-neutral-900">04</p>
            </div>
          </div>
        </div>

        <div class="p-2">
          <p class="px-4 py-2 text-xs font-bold uppercase tracking-widest text-neutral-400 mb-2">Titik PJU - Jl. Proklamasi</p>
          <div 
            v-for="item in SAMPLE_DATA" 
            :key="item.id"
            @click="selectItem(item)"
            class="p-4 rounded-xl cursor-pointer transition-all border mb-2 group"
            :class="selectedPJU?.id === item.id 
              ? 'bg-neutral-900 border-neutral-900 text-white shadow-lg' 
              : 'bg-white border-transparent hover:bg-neutral-50 hover:border-neutral-200'"
          >
            <div class="flex justify-between items-start mb-2">
              <p class="text-[10px] font-bold uppercase py-0.5 px-2 rounded-full"
                :class="selectedPJU?.id === item.id ? 'bg-white/20 text-white' : 'bg-neutral-100 text-neutral-600'"
              >
                {{ item.id }}
              </p>
              <Lightbulb 
                :class="[
                  'w-4 h-4', 
                  item.condition === 'Baik' ? 'text-emerald-500' : (selectedPJU?.id === item.id ? 'text-white' : 'text-neutral-900')
                ]" 
                :fill="item.condition === 'Baik' ? 'currentColor' : 'none'"
              />
            </div>
            <h3 class="font-bold text-sm leading-tight mb-1">{{ item.location }}</h3>
            <p class="text-xs line-clamp-1 opacity-70"
              :class="selectedPJU?.id === item.id ? 'text-neutral-300' : 'text-neutral-500'"
            >
              {{ item.condition === 'Rusak' ? (item.issue || 'Permasalahan Teknis') : 'Operasional Normal' }}
            </p>
          </div>
        </div>
      </div>
    </aside>

    <!-- --- MAP SURFACE --- -->
    <main class="flex-1 w-full h-full relative z-10">
      <div id="map-container" class="absolute inset-0 w-full h-full"></div>

      <!-- --- TOP BAR --- -->
      <div class="absolute top-4 left-4 right-4 flex justify-between items-center pointer-events-none z-20">
        <div class="bg-white/90 backdrop-blur-md px-4 py-2.5 rounded-full border border-neutral-100 shadow-md flex items-center gap-2 pointer-events-auto max-w-[85%] sm:max-w-none">
          <span class="w-1.5 h-1.5 bg-emerald-500 rounded-full animate-pulse shrink-0" />
          <p class="text-[10px] md:text-xs font-bold uppercase tracking-tight text-neutral-700 truncate">Jl. Proklamasi - PJU Monitor Live</p>
          <div class="h-3 w-[1px] bg-neutral-300 shrink-0" />
          <p class="text-[9px] md:text-[10px] font-mono text-neutral-500 shrink-0">Kuantan Tengah</p>
        </div>
      </div>

      <!-- --- FLOATING FLOOP CARD LIST TOGGLE FOR MOBILE --- -->
      <div class="absolute bottom-6 left-1/2 -translate-x-1/2 z-20 md:hidden pointer-events-none">
        <button 
          @click="isSidebarOpen = !isSidebarOpen"
          class="pointer-events-auto bg-neutral-900 border border-neutral-800 text-white font-bold px-6 py-3.5 rounded-full flex items-center gap-2.5 shadow-xl hover:bg-neutral-800 transition-all active:scale-95 text-xs tracking-wider uppercase"
        >
          <LayoutDashboard class="w-4 h-4 text-amber-500 animate-bounce" />
          <span>{{ isSidebarOpen ? 'Tutup Daftar' : 'Daftar PJU (11)' }}</span>
        </button>
      </div>
      
      <!-- --- DETAIL PANEL (BOTTOM SHEET on Mobile / SLIDE PANEL on Desktop) --- -->
      <Transition
        enter-active-class="transform transition ease-out duration-300"
        enter-from-class="translate-y-full md:translate-y-0 md:translate-x-full"
        enter-to-class="translate-y-0 md:translate-x-0"
        leave-active-class="transform transition ease-in duration-300"
        leave-from-class="translate-y-0 md:translate-x-0"
        leave-to-class="translate-y-full md:translate-y-0 md:translate-x-full"
      >
        <div 
          v-if="selectedPJU"
          class="absolute bottom-0 md:top-0 right-0 h-[80vh] md:h-full w-full md:w-[400px] bg-white border-t md:border-t-0 md:border-l border-neutral-200 shadow-2xl z-40 flex flex-col rounded-t-3xl md:rounded-t-none"
        >
          <!-- Pull-down decorative handle for mobile -->
          <div class="w-12 h-1 bg-neutral-200 rounded-full mx-auto my-3 md:hidden shrink-0"></div>

          <div class="px-6 pb-4 md:pt-6 border-b border-neutral-100 flex items-center justify-between h-14 md:h-16 shrink-0">
            <div class="overflow-hidden">
              <p class="text-[10px] font-bold text-neutral-400 uppercase tracking-[0.2em] mb-0.5">Tiang ID: {{ selectedPJU.id }}</p>
              <h2 class="text-base md:text-lg font-bold italic tracking-tight truncate">{{ selectedPJU.location }}</h2>
            </div>
            <button 
              @click="selectedPJU = null"
              class="p-2 hover:bg-neutral-100 rounded-full transition-colors shrink-0"
            >
              <X class="w-5 h-5" />
            </button>
          </div>

          <div class="flex-1 overflow-y-auto">
            <div class="aspect-video w-full relative group shrink-0">
              <img 
                :src="selectedPJU.image" 
                :alt="selectedPJU.id"
                class="w-full h-full object-cover"
              />
              <div class="absolute inset-0 bg-gradient-to-t from-black/60 to-transparent opacity-100 flex items-end p-4">
                <div class="flex items-center gap-2 text-white">
                  <Camera class="w-4 h-4" />
                  <span class="text-xs font-medium italic">Kondisi Terkini - {{ selectedPJU.reporter }}</span>
                </div>
              </div>
            </div>

            <div class="p-6 text-neutral-800">
              <div class="flex items-center gap-4 mb-6">
                <div class="flex-1">
                  <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Status Sistem</p>
                  <div class="flex items-center gap-2 p-2 rounded-lg border transition-colors"
                    :class="selectedPJU.condition === 'Baik' ? 'bg-emerald-50 border-emerald-100 text-emerald-700' : 'bg-neutral-100 border-neutral-200 text-neutral-900'"
                  >
                    <Lightbulb class="w-4 h-4" :fill="selectedPJU.condition === 'Baik' ? 'currentColor' : 'none'" />
                    <span class="text-xs md:text-sm font-bold">{{ selectedPJU.condition === 'Baik' ? 'PENERANGAN AKTIF' : 'PADAM / RUSAK' }}</span>
                  </div>
                </div>
                <div class="flex-1">
                  <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Status Pekerjaan</p>
                  <div class="flex items-center gap-2 bg-neutral-50 p-2 rounded-lg border border-neutral-100">
                     <span class="text-xs md:text-sm font-bold text-neutral-700">{{ selectedPJU.status }}</span>
                  </div>
                </div>
              </div>

              <div class="space-y-6">
                <section v-if="selectedPJU.condition === 'Rusak'">
                  <h4 class="flex items-center gap-2 text-xs font-bold uppercase text-neutral-900 mb-2">
                    <AlertTriangle class="w-3.5 h-3.5 text-amber-500" />
                    Analisis Kerusakan
                  </h4>
                  <p class="text-neutral-700 leading-relaxed font-serif italic text-base md:text-lg bg-neutral-50 p-4 rounded-xl border border-neutral-200">
                    "{{ selectedPJU.issue }}"
                  </p>
                </section>

                <div class="h-[1px] bg-neutral-100" />

                <section class="grid grid-cols-2 gap-4">
                  <div>
                    <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Tipe Lampu</p>
                    <p class="text-sm font-bold">{{ selectedPJU.type }}</p>
                  </div>
                  <div>
                    <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Maintenence Terakhir</p>
                    <p class="text-sm font-bold">{{ selectedPJU.lastMaintenance }}</p>
                  </div>
                </section>

                <section>
                  <p class="text-[10px] font-bold text-neutral-400 uppercase mb-1">Koordinat GPS</p>
                  <code class="text-[10px] flex items-center gap-1 bg-neutral-900 text-white p-2.5 rounded-md font-mono">
                    {{ selectedPJU.coordinates[0].toFixed(6) }}, {{ selectedPJU.coordinates[1].toFixed(6) }}
                  </code>
                </section>
              </div>
            </div>
          </div>

          <div class="p-6 border-t border-neutral-100 shrink-0 bg-white">
            <button class="w-full bg-amber-500 text-white font-bold py-3.5 rounded-2xl flex items-center justify-center gap-2 hover:bg-amber-600 transition-colors shadow-md text-sm">
              <span>ORDER PERBAIKAN</span>
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
