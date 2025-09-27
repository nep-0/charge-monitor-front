<template>
  <div v-if="station.layout" class="layout-map">
    <div class="layout-container" :style="containerStyle">
      <img
        :src="station.layout.imageUrl"
        :alt="`${station.title} 布局图`"
        class="layout-image"
        @load="onImageLoad"
      />
      <div
        v-for="(position, index) in station.layout.outletPositions"
        :key="position.index"
        class="outlet-dot"
        :class="getOutletClass(station.outlets[index])"
        :style="getMarkerStyle(position)"
        :title="getOutletTooltip(station.outlets[index], index + 1)"
      >
      </div>
    </div>
    <div class="layout-legend">
      <div class="legend-item">
        <div class="legend-marker available"></div>
        <span>可用</span>
      </div>
      <div class="legend-item">
        <div class="legend-marker occupied"></div>
        <span>使用中</span>
      </div>
      <div class="legend-item">
        <div class="legend-marker loading"></div>
        <span>加载中</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'OutletLayoutMap',
  props: {
    station: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      scale: 1,
      containerWidth: 0,
      containerHeight: 0,
      imageLoaded: false,
      updateInterval: null
    }
  },
  computed: {
    containerStyle() {
      const parentElement = this.$el?.parentElement;
      const availableWidth = parentElement ?
        Math.min(parentElement.offsetWidth - 40, 600) :
        Math.min(window.innerWidth - 80, 600);

      const originalWidth = this.station.layout.width;

      if (originalWidth <= availableWidth) {
        this.scale = 1;
        this.containerWidth = originalWidth;
        this.containerHeight = this.station.layout.height;
      } else {
        this.scale = availableWidth / originalWidth;
        this.containerWidth = availableWidth;
        this.containerHeight = this.station.layout.height * this.scale;
      }

      return {
        width: this.containerWidth + 'px',
        height: this.containerHeight + 'px'
      };
    }
  },
  watch: {
    station: {
      handler() {
        // Only recalculate when station data changes
        this.updateScale();
      }
    }
  },
  mounted() {
    console.log('OutletLayoutMap mounted');
    this.imageLoaded = true;
    this.updateScale();
    window.addEventListener('resize', this.updateScale);

    // Update scale every second
    this.updateInterval = setInterval(() => {
      this.updateScale();
    }, 1000);
  },
  beforeDestroy() {
    console.log('OutletLayoutMap beforeDestroy, clearing interval:', this.updateInterval);
    window.removeEventListener('resize', this.updateScale);
    if (this.updateInterval) {
      clearInterval(this.updateInterval);
    }
  },
  methods: {
    updateScale() {
      // Force recalculation of container dimensions
      if (this.$el) {
        const parentElement = this.$el.parentElement;
        const availableWidth = parentElement ?
          Math.min(parentElement.offsetWidth - 40, 600) :
          Math.min(window.innerWidth - 80, 600);

        const originalWidth = this.station.layout.width;

        if (originalWidth <= availableWidth) {
          this.scale = 1;
          this.containerWidth = originalWidth;
          this.containerHeight = this.station.layout.height;
        } else {
          this.scale = availableWidth / originalWidth;
          this.containerWidth = availableWidth;
          this.containerHeight = this.station.layout.height * this.scale;
        }
      }

      this.$forceUpdate();
    },
    onImageLoad() {
      this.imageLoaded = true;
      this.$nextTick(() => {
        this.updateScale();
      });
    },
    getMarkerStyle(position) {
      const dotSize = Math.max(8, 12 * this.scale); // Minimum 8px, scales with container
      return {
        left: (position.x * this.scale) + 'px',
        top: (position.y * this.scale) + 'px',
        width: dotSize + 'px',
        height: dotSize + 'px'
      };
    },
    getOutletClass(outlet) {
      if (!outlet || outlet.power === undefined) return 'loading';
      return !outlet.power ? 'available' : 'occupied';
    },
    getOutletStatus(outlet) {
      if (!outlet || outlet.power === undefined) return '加载中';
      return !outlet.power ? '可用' : `使用中 (${outlet.used_minutes}分钟)`;
    },
    getOutletTooltip(outlet, number) {
      if (!outlet || outlet.power === undefined) {
        const name = outlet?.name ? `${outlet.name}` : `${number}`;
        return `插座 ${name}: 加载中...`;
      }
      const name = outlet.name ? `${outlet.name}` : `${number}`;
      if (!outlet.power) {
        return `插座 ${name}: 可用`;
      }
      return `插座 ${name}: ${outlet.power} · ${outlet.used_minutes}分钟`;
    },
    getDisplayPower(outlet) {
      if (!outlet || outlet.power === undefined) return '';
      return outlet.power || '可用';
    }
  }
}
</script>

<style scoped>
.layout-map {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.layout-container {
  position: relative;
  border: 2px solid #e1e5e9;
  border-radius: 8px;
  overflow: hidden;
  background-color: #f8f9fa;
  max-width: 100%;
}

.layout-image {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: contain;
  object-position: top left;
}

.outlet-dot {
  position: absolute;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  cursor: pointer;
  transform: translate(-50%, -50%);
  border: 2px solid white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.3);
}

.outlet-dot:hover {
  transform: translate(-50%, -50%) scale(1.3);
  z-index: 10;
  box-shadow: 0 3px 6px rgba(0,0,0,0.4);
}

.outlet-dot.available {
  background-color: #4caf50;
}

.outlet-dot.occupied {
  background-color: #f44336;
}

.outlet-dot.loading {
  background-color: #ff9800;
}

@media (max-width: 768px) {
  .outlet-dot {
    width: 10px;
    height: 10px;
    border-width: 1px;
  }
}

@media (max-width: 480px) {
  .outlet-dot {
    width: 8px;
    height: 8px;
  }

  .layout-legend {
    flex-wrap: wrap;
    justify-content: center;
    gap: 0.5rem !important;
  }

  .legend-item {
    font-size: 0.7rem !important;
  }
}

.layout-legend {
  display: flex;
  gap: 1rem;
  margin-top: 0.5rem;
  padding: 0.5rem;
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 4px;
  border: 1px solid #e1e5e9;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 0.25rem;
  font-size: 0.8rem;
}

.legend-marker {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 1px solid white;
}

.legend-marker.available {
  background-color: #4caf50;
}

.legend-marker.occupied {
  background-color: #f44336;
}

.legend-marker.loading {
  background-color: #ff9800;
}
</style>