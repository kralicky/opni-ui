<script>
import AsyncButton from '@/components/AsyncButton';
import Card from '@/components/Card';
import { exceptionToErrorsArray } from '@/utils/error';
import LabeledInput from '@/components/form/LabeledInput';
import KeyValue from '@/components/form/KeyValue';
import { updateCluster } from '@/product/opni/utils/requests';

export default {
  components: {
    Card,
    KeyValue,
    LabeledInput,
    AsyncButton,
  },
  data() {
    return {
      id:     '',
      name:   '',
      labels: {},
      errors: [],
    };
  },
  methods: {
    close() {
      this.$modal.hide('edit-cluster-dialog');
    },

    open(cluster) {
      this.$set(this, 'id', cluster.id);
      this.$set(this, 'name', cluster.name);
      this.$set(this, 'labels', cluster.labels);
      this.$modal.show('edit-cluster-dialog');
    },

    async save(buttonDone) {
      try {
        await updateCluster(this.id, this.name, this.labels);
        buttonDone(true);
        this.$emit('save');
        this.close();
      } catch (err) {
        this.errors = exceptionToErrorsArray(err);
        buttonDone(false);
      }
    },
  },
};
</script>

<template>
  <modal
    ref="editClusterDialog"
    class="edit-cluster-dialog"
    name="edit-cluster-dialog"
    styles="background-color: var(--nav-bg); border-radius: var(--border-radius); max-height: 90vh;"
    height="auto"
    width="600"
    :scrollable="true"
    @closed="close()"
  >
    <Card
      class="prompt-restore"
      :show-highlight-border="false"
      title="Edit Cluster"
    >
      <h4 ref="clusterLabel" slot="title" class="text-default-text pt-4" v-html="`Edit Cluster: ${ id }`" />
      <div slot="body" class="pt-10">
        <div class="row mb-10">
          <div class="col span-12">
            <LabeledInput v-model.trim="name" label="Name (optional)" />
          </div>
        </div>
        <div class="row">
          <KeyValue
            v-model="labels"
            mode="edit"
            :read-allowed="false"
            :value-multiline="false"
            label="Labels"
            add-label="Add Label"
          />
        </div>
      </div>

      <div slot="actions" class="buttons">
        <button class="btn role-secondary mr-10" @click="close">
          {{ t("generic.cancel") }}
        </button>

        <AsyncButton mode="edit" @click="save" />
      </div>
    </Card>
  </modal>
</template>
<style lang='scss' scoped>
.prompt-restore {
  margin: 0;
}
.buttons {
  display: flex;
  justify-content: flex-end;
  width: 100%;
}

.edit-cluster-dialog {
  border-radius: var(--border-radius);
  overflow-y: scroll;
  max-height: 100vh;
  & ::-webkit-scrollbar-corner {
    background: rgba(0, 0, 0, 0);
  }
}
</style>
