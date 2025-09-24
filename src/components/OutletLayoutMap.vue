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
        class="outlet-marker"
        :class="getOutletClass(station.outlets[index])"
        :style="getMarkerStyle(position)"
        :title="getOutletTooltip(station.outlets[index], index + 1)"
      >
        <div class="marker-content">
          <div v-if="station.outlets[index] && station.outlets[index].power !== undefined" class="marker-info">
            <div class="power-info">{{ getDisplayPower(station.outlets[index]) }}</div>
            <div v-if="station.outlets[index].power" class="duration-info">{{ station.outlets[index].used_minutes }}min</div>
          </div>
        </div>
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
      containerHeight: 0
    }
  },
  computed: {
    containerStyle() {
      const maxWidth = Math.min(this.station.layout.width, window.innerWidth - 80);
      this.scale = maxWidth / this.station.layout.width;

      this.containerWidth = this.station.layout.width * this.scale;
      this.containerHeight = this.station.layout.height * this.scale;

      return {
        width: this.containerWidth + 'px',
        height: this.containerHeight + 'px',
        transform: `scale(${this.scale})`,
        transformOrigin: 'top left'
      };
    }
  },
  mounted() {
    this.updateScale();
    window.addEventListener('resize', this.updateScale);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.updateScale);
  },
  methods: {
    updateScale() {
      this.$forceUpdate();
    },
    onImageLoad() {
      this.$forceUpdate();
    },
    getMarkerStyle(position) {
      return {
        left: (position.x * this.scale) + 'px',
        top: (position.y * this.scale) + 'px'
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
        return `插座 ${number}: 加载中...`;
      }
      if (!outlet.power) {
        return `插座 ${number}: 可用`;
      }
      return `插座 ${number}: ${outlet.power}kW · ${outlet.used_minutes}分钟`;
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
}

.outlet-marker {
  position: absolute;
  min-width: 36px;
  min-height: 36px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transform: translate(-50%, -50%);
  border: 2px solid white;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
  padding: 2px;
}

@media (max-width: 768px) {
  .outlet-marker {
    min-width: 28px;
    min-height: 28px;
    border-width: 1px;
  }

  .power-info {
    font-size: 7px !important;
  }

  .duration-info {
    font-size: 6px !important;
  }
}

@media (max-width: 480px) {
  .outlet-marker {
    min-width: 24px;
    min-height: 24px;
  }

  .power-info {
    font-size: 6px !important;
  }

  .duration-info {
    font-size: 5px !important;
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

.marker-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.marker-number {
  font-size: 10px;
  font-weight: bold;
  color: white;
  line-height: 1;
}

.marker-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 1px;
}

.power-info {
  font-size: 8px;
  font-weight: bold;
  color: white;
  line-height: 1;
}

.duration-info {
  font-size: 7px;
  color: rgba(255, 255, 255, 0.9);
  line-height: 1;
}

.outlet-marker.available {
  background-color: #4caf50;
}

.outlet-marker.occupied {
  background-color: #f44336;
}

.outlet-marker.loading {
  background-color: #ff9800;
}

.outlet-marker:hover {
  transform: translate(-50%, -50%) scale(1.2);
  z-index: 10;
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