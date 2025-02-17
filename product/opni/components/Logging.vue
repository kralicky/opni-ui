<script>
import day from 'dayjs';
import { getStartTime } from '@/product/opni/utils/requests';
import InsightsChart from './InsightsChart';
import Configurator, { DEFAULT_CONFIGURATION } from './Configurator';
import Breakdown from './Breakdown';

export default {
  components: {
    Breakdown, Configurator, InsightsChart
  },

  data() {
    return {
      areasOfInterest:       [],
      config:                { ...DEFAULT_CONFIGURATION },
      selection:             null,
      potentialSelection:    null,
      now:                   day(),
      refreshRateInterval:   null,
    };
  },

  created() {
    this.createInterval();
  },

  beforeDestroy() {
    this.deleteInterval();
  },

  computed: {
    defaultSelection() {
      return {
        from: getStartTime(this.now, this.config.range, this.config.granularity),
        to:   this.now
      };
    }
  },

  methods: {
    regionDrag(region) {
      this.$set(this, 'potentialSelection', region);
    },
    regionSelected(region) {
      this.$set(this, 'potentialSelection', null);
      this.$set(this, 'selection', region);
    },
    deleteInterval() {
      if (this.refreshRateInterval) {
        clearInterval(this.refreshRateInterval);
        this.$set(this, 'refreshRateInterval', null);
      }
    },
    createInterval() {
      this.deleteInterval();
      const ms = this.now.add(this.config.refreshRate.count, this.config.refreshRate.unit).diff(this.now);

      this.$set(this, 'refreshRateInterval', setInterval(this.refresh, ms));
    },
    refresh() {
      this.$set(this, 'now', day());
    }
  },

  watch: {
    'config.refreshRate'() {
      this.createInterval();
    }
  }
};
</script>
<template>
  <div>
    <div class="bar">
    </div>
    <Configurator v-model="config" />
    <InsightsChart
      :now="now"
      :granularity="config.granularity"
      :range="config.range"
      :selection="potentialSelection || selection"
      @areaOfInterestSelected="regionSelected"
      @regionDrag="regionDrag"
      @regionSelected="regionSelected"
    />
    <Breakdown
      :potential-selection="potentialSelection"
      :selection="selection || defaultSelection"
    />
  </div>
</template>

<style lang="scss" scoped>
::v-deep {
  .bubble {
    &.workload {
      border-color: var(--app-other-accent);

      &::before {
        background-color: var(--app-other-accent);
      }
    }

    &.control-plane {
      border-color: var(--app-rancher-accent);

      &::before {
        background-color: var(--app-rancher-accent);
      }
    }
  }
}

.card {
  margin: 0;
}

.bar {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

img {
  $size: 20px;
  width: $size;
  height: $size;
}
</style>
