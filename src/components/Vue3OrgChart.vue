<script setup lang="ts">
import type {IApi, IPanzoom, IProps} from "@/utils/types";
import {provide, watch} from 'vue';
import Node from './Node.vue';

// props
const props = withDefaults(defineProps<IProps>(), {
  data: () => [],
  minimap: false
})

// Panzoom setup, provide it to the child components
import {usePanzoom} from "@/composables/usePanzoom";

const {instance, scene, container} = usePanzoom() as IPanzoom;
provide('panzoom', {instance, scene, container} as IPanzoom);

// Data setup, provide data and loading state to the child components
import {useData} from "@/composables/useData";

const {data, loading} = useData({initialData: props.data, json: props.json});
provide('content', {data, loading});

// Api setup, useful functions to interact with the org chart
import {useApi} from "@/composables/useApi";
import MiniMap from "@/components/MiniMap.vue";

const api: IApi = useApi(instance, data, container, scene, props.minimap);
provide('api', api);

//  emit event when data is loaded and ready, provide api object
const emit = defineEmits(['onReady']);
watch(() => loading.value, (loadingState) => {
  if (!loadingState) {
    emit('onReady', {api})
  }
});

</script>

<template>
  <div class="vue3-org-chart">
    <div ref="container" class="vue3-org-chart-container">
      <div ref="scene" class="vue3-org-chart-scene">
        <Node v-if="data.length && api.rootId()" :id="api.rootId()" key="root">
          <template #node="params">
            <slot name="node" v-bind="params"/>
          </template>
        </Node>
        <div v-else>
            <span v-if="loading">
              <slot name="loading">Loading...</slot>
            </span>
          <span v-else>
              <slot name="no-data">No data</slot>
            </span>
        </div>
      </div>
    </div>
    <MiniMap v-if="!loading && api.minimap.state" />
  </div>
</template>
