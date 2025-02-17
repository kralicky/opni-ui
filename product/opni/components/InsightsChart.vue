<script>
import Card from '@/components/Card';
import { formatForTimeseries, findBucket, showTooltip } from '@/product/opni/utils/munging';
import TimeSeries from '@/components/graph/TimeSeries';
import day from 'dayjs';
import Loading from '@/components/Loading';
import { getAreasOfInterest, getOverallBreakdownSeries } from '@/product/opni/utils/requests';

export default {
  components: {
    Card, Loading, TimeSeries
  },

  props: {
    now: {
      type:     Object,
      required: true
    },
    selection: {
      type:    Object,
      default: null
    },
    granularity: {
      type:     Object,
      required: true
    },
    range: {
      type:     Object,
      required: true
    }
  },

  fetch() {
    this.load();
  },

  data() {
    return {
      areasOfInterest: [],
      insights:        [],
      loading:         true
    };
  },

  computed: {
    insightSeries() {
      const out = this.formatForTimeseries(this.insights);

      out['Anomalous'].color = 'var(--error)';
      out['Normal'].color = 'var(--primary-low-opacity)';
      out['Suspicious'].color = 'var(--warning)';

      return out;
    },
    invertedRegions() {
      if (!this.selection) {
        return [];
      }

      const targets = [this.selection];
      const converted = targets.flatMap(({ from, to }) => [from.valueOf(), to.valueOf()]);
      const eachPoint = [this.insightSeries?.timestamp?.data?.[0] || 0, ...converted, day().valueOf()];
      const result = [];

      for (let i = 0; i < eachPoint.length - 1; i += 2) {
        const j = i + 1;

        result.push({
          start: eachPoint[i],
          end:   eachPoint[j],
          axis:  'x',
          class: 'inverted-region'
        });
      }

      return result;
    },
    topBorderRegions() {
      return this.areasOfInterest.map(({ from, to }) => ({
        start: from.valueOf(),
        end:   to.valueOf(),
        axis:  'x',
        class: 'top-border-region'
      }));
    },
    regions() {
      return [...this.invertedRegions, ...this.topBorderRegions];
    }
  },

  methods: {
    formatForTimeseries,
    findBucket,
    showTooltip,
    mapTimeSeries(data, columns) {
      const index = data.index;
      const fromIndex = index - 1;
      const toIndex = index + 1;
      const timestamps = columns[0];

      return {
        fromTo: {
          from:  fromIndex >= 0 ? day(timestamps[fromIndex]) : null,
          to:    toIndex < timestamps.length ? day(timestamps[toIndex]) : null,
        },
        value: data.value,
        line:  data.id,
        index
      };
    },

    moveTabs() {
      const container = document.querySelector('.card-container');
      const containerRect = container.getBoundingClientRect();
      const borderRegions = document.querySelectorAll('.top-border-region');
      const tabs = document.querySelectorAll('.areas-of-interest-tab');

      for (let i = 0; i < tabs.length; i += 1) {
        const border = borderRegions[i];
        const tab = tabs[i];
        const borderRect = border.getBoundingClientRect();

        const left = borderRect.left - containerRect.left;
        const top = borderRect.top - containerRect.top;
        const width = borderRect.width;

        tab.style.left = `${ left }px`;
        tab.style.top = `${ top }px`;
        tab.style.width = `${ width }px`;
      }
    },

    selectAreaOfInterest(areaOfInterest) {
      this.$emit('areaOfInterestSelected', areaOfInterest);
    },

    async load() {
      this.loading = true;

      const responses = await Promise.all([
        getOverallBreakdownSeries(this.now, this.range, this.granularity),
        getAreasOfInterest(this.now, this.range, this.granularity)
      ]);

      [
        this.insights,
        this.areasOfInterest,
      ] = responses;

      this.loading = false;
    }
  },

  watch: {
    now() {
      this.load();
    },

    range() {
      this.load();
    },

    granularity() {
      this.load();
    }
  }
};
</script>
<template>
  <div>
    <Card class="card mt-10" :show-actions="false" :show-highlight-border="false">
      <template #body>
        <Loading v-if="loading" />
        <div v-else>
          <TimeSeries
            ref="insights"
            class="timeseries mt-10 mb-10"
            chart-id="insights"
            :colors="{'Anomalous':'var(--error)', 'Normal': 'var(--primary)', 'Suspicious': 'var(--warning)'}"
            x-key="timestamp"
            :data-series="insightSeries"
            :regions="regions"
            v-on="$listeners"
            @onRendered="moveTabs"
          >
            <template #legend="api">
              <div class="custom-legend mt-10">
                <div class="left">
                  <div class="datum" @mouseenter="api.focus('Anomalous')" @mouseleave="api.revert()" @click="api.toggle('Anomalous')">
                    <div class="square anomalous mr-5"></div><label>{{ t('opni.chart.labels.anomaly') }}</label>
                  </div>
                  <div class="datum" @mouseenter="api.focus('Suspicious')" @mouseleave="api.revert()" @click="api.toggle('Suspicious')">
                    <div class="square suspicious mr-5"></div><label>{{ t('opni.chart.labels.suspicious') }}</label>
                  </div>
                </div>
                <div class="right">
                  <div class="datum" @mouseenter="api.focus('Normal')" @mouseleave="api.revert()" @click="api.toggle('Normal')">
                    <label>{{ t('opni.chart.labels.normal') }}</label><div class="square normal ml-5"></div>
                  </div>
                </div>
              </div>
            </template>
          </TimeSeries>
          <div v-for="(aoi, i) in areasOfInterest" :key="i" v-tooltip="t('opni.chart.areaOfInterest')" class="areas-of-interest-tab" @click="selectAreaOfInterest(aoi)" />
        </div>
      </template>
    </Card>
  </div>
</template>

<style lang="scss" scoped>
::v-deep {
  .card-body {
    height: 382px;
  }

  .card-container {
    position: relative;
  }

  .loading-indicator .overlay {
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
  }

  .inverted-region {
    opacity: 0.7;
    fill: #DDD;
  }

  .top-border-region {
    fill: none;
  }
}

.areas-of-interest-tab {
  height: 15px;

  position: absolute;
  cursor: pointer;

  &:before {
    position: absolute;
    left: 0;
    right: 0;
    top: 8px;

    height: 5px;
    background-color: var(--error);
    opacity: 0.6;
    content: ' ';
  }

  &:hover:before {
    height: 10px;
  }
}

.custom-legend {
  display: flex;

  flex-direction: row;
  justify-content: space-between;

  .datum {
    display: flex;

    flex-direction: row;
    align-items: center;

    &:hover {
      .square {
        opacity: 0.5;
      }
    }

    &, label {
      cursor: pointer;
    }
  }

  .left {
    padding-left: 50px;
  }

  .right {
    padding-right: 40px;
  }

  .square {
    $size: 12px;

    display: inline-block;

    width: $size;
    height: $size;

    &.anomalous {
      background-color: var(--error);
    }

    &.suspicious {
      background-color: var(--warning);
    }

    &.normal {
      background-color: var(--primary);
    }
  }
}

.card {
  margin: 0;
}
</style>
