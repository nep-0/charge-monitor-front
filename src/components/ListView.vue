<template>
  <div class="list-view">
    <Message severity="info" class="fav-info-banner" :closable="true">
      <div class="fav-info-content">
        <i class="pi pi-heart-fill"></i>
        <span>点击爱心图标以收藏。收藏的充电站将出现在列表的顶部。</span>
      </div>
    </Message>

    <div class="search-container">
      <IconField iconPosition="left">
        <InputIcon class="pi pi-search" />
        <InputText
          v-model="searchQuery"
          placeholder="搜索充电站..."
          class="search-input"
        />
      </IconField>
    </div>

    <Panel v-for="station in filteredStations" :key="station.title" toggleable :collapsed="true" ref="panels">
      <template #header>
        <div class="station-header">
          <div class="station-info">
            <Tag
              :value="`${getAvailableCount(station)}\/${station.outlets.length}`"
              :severity="getAvailableCount(station) > 0 ? 'success' : 'danger'"
              class="availability-tag"
            />
            <span class="station-title" @click="togglePanel(station.title)">{{ station.title }}</span>
          </div>
          <Button
            :icon="isFavorite(station.title) ? 'pi pi-heart-fill' : 'pi pi-heart'"
            class="p-button-text favorite-btn"
            :class="{ 'favorite-active': isFavorite(station.title) }"
            @click="toggleFavorite(station.title)"
            size="large"
          />
        </div>
      </template>
      <div class="outlet-grid">
        <Outlet
          v-for="outlet in station.outlets"
          :key="outlet.id"
          :outlet="outlet"
        />
      </div>
    </Panel>
  </div>
</template>

<script>
import axios from 'axios';
import Panel from 'primevue/panel';
import Button from 'primevue/button';
import Message from 'primevue/message';
import Tag from 'primevue/tag';
import InputText from 'primevue/inputtext';
import IconField from 'primevue/iconfield';
import InputIcon from 'primevue/inputicon';
import Outlet from './Outlet.vue';
import { stationMap } from '../station-map.js';

export default {
  name: 'ListView',
  components: {
    Panel,
    Button,
    Message,
    Tag,
    InputText,
    IconField,
    InputIcon,
    Outlet
  },
  data() {
    return {
      stations: [],
      favorites: new Set(),
      searchQuery: '',
      panelStates: {},
      polling: null
    }
  },
  computed: {
    filteredStations() {
      const filtered = this.searchQuery
        ? this.stations.filter(station =>
            station.title.toLowerCase().includes(this.searchQuery.toLowerCase())
          )
        : this.stations;

      return [...filtered].sort((a, b) => {
        const aIsFav = this.isFavorite(a.title);
        const bIsFav = this.isFavorite(b.title);
        if (aIsFav && !bIsFav) return -1;
        if (!aIsFav && bIsFav) return 1;
        return 0;
      });
    }
  },
  methods: {
    async fetchData() {
      try {
        const response = await axios.get('https://gokwmudxqjkl.sealosgzg.site/outlets');
        const outlets = response.data;
        this.stations = Object.keys(stationMap).map(stationName => {
          const stationOutlets = stationMap[stationName];
          return {
            title: stationName,
            outlets: stationOutlets.map(outletId => {
              return {
                id: outletId,
                ...outlets[outletId]
              };
            })
          };
        });
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    },
    loadFavorites() {
      const saved = localStorage.getItem('stationFavorites');
      if (saved) {
        this.favorites = new Set(JSON.parse(saved));
      }
    },
    saveFavorites() {
      localStorage.setItem('stationFavorites', JSON.stringify([...this.favorites]));
    },
    isFavorite(stationTitle) {
      return this.favorites.has(stationTitle);
    },
    toggleFavorite(stationTitle) {
      if (this.favorites.has(stationTitle)) {
        this.favorites.delete(stationTitle);
      } else {
        this.favorites.add(stationTitle);
      }
      this.saveFavorites();
    },
    getAvailableCount(station) {
      return station.outlets.filter(outlet => !outlet.power && outlet.power !== undefined).length;
    },
    togglePanel(stationTitle) {
      const panelIndex = this.filteredStations.findIndex(station => station.title === stationTitle);
      if (panelIndex >= 0 && this.$refs.panels && this.$refs.panels[panelIndex]) {
        this.$refs.panels[panelIndex].toggle();
      }
    }
  },
  created() {
    this.loadFavorites();
    this.fetchData();
    this.polling = setInterval(this.fetchData, 10000);
  },
  beforeDestroy() {
    clearInterval(this.polling);
  }
}
</script>

<style scoped>
.list-view {
  width: 85%;
  max-width: 1200px;
  margin: 0 auto;
}

.fav-info-banner {
  margin-bottom: 1rem;
}

.fav-info-content {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.fav-info-content i {
  color: #e74c3c;
}

.search-container {
  margin-bottom: 1rem;
}

.search-input {
  width: 100%;
}

.station-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.station-info {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.availability-tag {
  font-weight: bold;
  font-size: 0.85rem;
}

.station-title {
  font-weight: bold;
  cursor: pointer;
  user-select: none;
}

.station-title:hover {
  color: #007bff;
}

.favorite-btn {
  padding: 0.25rem !important;
  margin-left: 0.5rem;
  color: #6c757d !important;
}

.favorite-btn.favorite-active {
  color: #e74c3c !important;
}

.favorite-btn:hover {
  color: #e74c3c !important;
  background-color: transparent !important;
}

.outlet-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-top: 1rem;
}

:deep(.p-panel-header) {
  background-color: #f1f1f1;
}

</style>
