<template>
  <panel-unit title="rendering">
    <a-form-item label="show">
      <a-row type="flex" :gutter="[8]">
        <a-col>
          <a-radio-group v-model:value="render.side">
            <a-radio-button value="top">Top</a-radio-button>
            <a-radio-button value="bottom">Bottom</a-radio-button>
          </a-radio-group>
        </a-col>
      </a-row>
    </a-form-item>
    <a-form-item label="solder mask">
      <a-row type="flex" :gutter="[8, 8]">
        <a-col>
          <a-select v-model:value="render.sm" :style="{ width: '10em' }">
            <a-select-option value="green">Green</a-select-option>
            <a-select-option value="red">Red</a-select-option>
            <a-select-option value="yellow">Yellow</a-select-option>
            <a-select-option value="blue">Blue</a-select-option>
            <a-select-option value="white">White</a-select-option>
            <a-select-option value="black">Black</a-select-option>
            <a-select-option value="purple">Purple</a-select-option>
          </a-select>
        </a-col>
      </a-row>
    </a-form-item>
    <a-form-item label="bare copper">
      <a-row type="flex" :gutter="[8]">
        <a-col>
          <a-radio-group v-model:value="render.cf">
            <a-radio-button value="tin">Tin</a-radio-button>
            <a-radio-button value="gold">Gold</a-radio-button>
          </a-radio-group>
        </a-col>
      </a-row>
    </a-form-item>
    <a-form-item label="Pad">
      <a-row type="flex" :gutter="[8]">
        <a-col>
          <a-checkbox v-model:checked="render.sp">
            Draw solder paste
          </a-checkbox>
        </a-col>
      </a-row>
    </a-form-item>
  </panel-unit>
</template>

<script lang="ts" setup>
import type { RenderOptions } from '@/utils/gerber';
import { defineProps, reactive } from 'vue';

import PanelUnit from '@/components/XPanelUnit.vue';

const props = defineProps<{
  render: RenderOptions;
}>();

const render = reactive(props.render);
</script>
