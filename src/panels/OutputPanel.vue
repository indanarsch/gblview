<template>
  <panel-unit title="Output options">
    <a-form-item label="Format">
      <a-row type="flex" :gutter="[8]">
        <a-col>
          <a-radio-group v-model:value="format">
            <a-radio-button value="svg">SVG</a-radio-button>
            <a-radio-button value="png">PNG</a-radio-button>
          </a-radio-group>
        </a-col>
      </a-row>
    </a-form-item>
    <a-form-item label="layered">
      <a-row type="flex" :gutter="[8]">
        <a-col>
          <a-checkbox v-model:checked="layers">
            Output layered file
          </a-checkbox>
        </a-col>
      </a-row>
    </a-form-item>
    <a-form-item label="relief">
      <a-row type="flex" :gutter="[8]">
        <a-col>
          <a-checkbox v-model:checked="relief">
            Export relief texture
          </a-checkbox>
          <a-tooltip placement="right" title="Can be used as relief texture map in 3D software such as Fusion 360">
            <info-circle-outlined />
          </a-tooltip>
        </a-col>
      </a-row>
    </a-form-item>
    <div>
      <a-button
        type="primary"
        :disabled="!props.gerber || props.layers.length == 0"
        :loading="rendering"
        @click="output">
        output file
      </a-button>
    </div>
  </panel-unit>
</template>

<script lang="ts" setup>
import { defineProps, ref } from 'vue';
import { InfoCircleOutlined } from '@ant-design/icons-vue';
import stackup, { type InputLayer } from 'pcb-stackup';
import { render } from 'gerber-to-svg';

import PanelUnit from '@/components/XPanelUnit.vue';

import { renderStack, type RenderOptions } from '@/utils/gerber';
import { toPNG, toSVG } from '@/utils/svg';
import { outputZip } from '@/utils/zip';

const props = defineProps<{
  gerber: File | undefined;
  layers: InputLayer[];
  render: RenderOptions;
}>();

const format = ref<'svg' | 'png'>('svg');
const layers = ref(false);
const relief = ref(false);

const rendering = ref(false);
async function output(): Promise<void> {
  rendering.value = true;

  const stack = await renderStack(props.layers, props.render);

  const files: Record<string, Uint8Array> = {};

  const ext = format.value;
  const write = ext == 'png' ? toPNG : toSVG;

  files[`top.${ext}`] = await write(stack.top.svg);
  files[`bottom.${ext}`] = await write(stack.bottom.svg);

  if (layers.value) {
    for (const layer of stack.layers) {
      files[`${layer.filename}.${ext}`] = await write(render(layer.converter, {}));
    }
  }

  if (relief.value) {
    const reliefStack = await stackup(props.layers, {
      color: {
        fr4: 'transparent',
        cu: '#ffffff',
        cf: 'transparent',
        sm: 'transparent',
        ss: 'transparent',
        sp: 'transparent',
        out: 'transparent',
      },
    });

    files[`top-relief.png`] = await toPNG(reliefStack.top.svg, '#000000');
    files[`bottom-relief.png`] = await toPNG(reliefStack.bottom.svg, '#000000');
  }

  await outputZip(files, `${basename(props.gerber!.name)}-img.zip`);

  rendering.value = false;
}

function basename(name: string): string{
  return name.replace(/\..+$/, '');
}
</script>
