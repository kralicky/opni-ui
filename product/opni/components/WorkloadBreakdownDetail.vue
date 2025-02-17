<script>
import SortableTable from '@/components/SortableTable';
import {
  SIMPLE_NAME, BREAKDOWN_RESOURCE, ANOMALY, NORMAL, SUSPICIOUS
} from '@/config/table-headers';
import LogsDrawer from './LogsDrawer';
import { WorkloadBreakdownAggregation } from '~/product/opni/models/overallBreakdown/WorkloadBreakdownAggregation';
import { getWorkloadLogs } from '~/product/opni/utils/requests';

export default {
  components: { LogsDrawer, SortableTable },

  props: {
    breakdown: {
      type:    WorkloadBreakdownAggregation,
      default: null,
    },

    fromTo: {
      type:     Object,
      required:  true,
    },
  },

  data() {
    return {
      headers: [
        {
          ...SIMPLE_NAME,
          width: null,
          value: 'breakdown.name'
        },
        BREAKDOWN_RESOURCE,
        ANOMALY(this.select, undefined, 'breakdown.insights.anomalyFormatted', 'breakdown.insights.anomaly'),
        SUSPICIOUS(this.select, undefined, 'breakdown.insights.suspiciousFormatted', 'breakdown.insights.suspicious'),
        NORMAL(undefined, undefined, 'breakdown.insights.normalFormatted', 'breakdown.insights.normal')
      ]
    };
  },

  computed: {
    rows() {
      return Object.entries(this.breakdown)
        .flatMap(([resource, resourceBreakdown]) => {
          return resourceBreakdown.map(breakdown => ({
            breakdown,
            resource
          }));
        });
    },
  },

  methods: {
    select(level, row) {
      this.$refs.drawer.open({ level, row });
    },
    async logGetter(level, row, scrollId) {
      return await getWorkloadLogs(this.fromTo.from, this.fromTo.to, level, row.breakdown.name, row.breakdown.namespace, row.breakdown.type, scrollId);
    }
  }
};
</script>
<template>
  <div>
    <SortableTable
      :rows="rows"
      group-by="breakdown.namespace"
      :headers="headers"
      :search="false"
      :table-actions="false"
      :row-actions="false"
      :paging="true"
      default-sort-by="anomaly"
      :default-sort-descending="true"
      key-field="id"
    />
    <LogsDrawer ref="drawer" :from-to="fromTo" :log-getter="logGetter" />
  </div>
</template>

<style lang="scss" scoped>
.detail {
  height: 100%;
}

.contents {
  overflow-y: scroll;
  height: 100%;
}

h1, h3 {
  display: inline-block;
}
</style>
