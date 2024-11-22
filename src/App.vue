<template>
  <Navbar />
  <div class="flex flex-col w-full py-10 default-container">
    <div class="mt-8 md:mt-16 md:mb-6 mb-6">
      <div class="md:my-14 md:mb-8">
        <div class="md:mb-12 text-center">
          <div class="text-5xl font-bold text-color w-full">
            Tailwind CSS
            <br />
            Color Palette Generator
          </div>
          <p class="text-zinc-500 font-thin mt-6 text-xl">
            Press
            <button
              class="inline-block border border-gray-500 px-1 text-base font-semibold rounded-[5px] h-[25px] hover:bg-zinc-800 transition duration-200 ease-in-out align-baseline"
              :class="{
                'bg-yellow-600 text-zinc-200 border-zinc-200': spacebarPress,
              }"
              @click="generateRandomColors()"
            >
              spacebar
            </button>
            , enter a hexcode or change the RGB values to generate a color
            palette.
          </p>
        </div>

        <div class="flex flex-col gap-5 max-w-lg mx-auto">
          <ColorPicker ref="colorPickerRef" @change="onColorChange($event)" />

          <template v-if="secondaryColorPalette !== null">
            <ColorPicker
              ref="secondaryColorPickerRef"
              @change="onSecondaryColorChange($event)"
            />
          </template>
        </div>

        <div class="justify-center mt-2 hidden md:flex">
          <button
            class="flex items-center gap-1 cursor-pointer text-color-muted-extra p-4"
            @click="addSecondaryColor()"
            v-if="secondaryColorPalette === null"
          >
            <Plus class="size-5" />
            Add secondary color
          </button>
        </div>
      </div>
    </div>

    <div class="flex flex-col gap-5">
      <ColorPalette :colorPalette="colorPalette" :colorName="colorName" />
      <ColorPalette2
        :colorPalette="secondaryColorPalette"
        :colorName="secondaryColorName"
        v-if="secondaryColorPalette !== null"
      />
    </div>
  </div>
  <Badges />
  <Buttons />
  <Authentication />
  <Spinners />
</template>

<script setup lang="ts">
  // native
  import { onMounted, onUnmounted } from 'vue';
  import { ref } from 'vue';
  import Badges from './components/Badges.vue';
  import Buttons from './components/Buttons.vue';
  import Spinners from './components/Spinners.vue';

  // components
  import ColorPicker from '@/components/ColorPicker.vue';
  import ColorPalette from '@/components/ColorPalette.vue';
  import ColorPalette2 from '@/components/ColorPalette2.vue';

  // third-party
  import tailwindcssPaletteGenerator from '@bobthered/tailwindcss-palette-generator';
  import { Plus } from 'lucide-vue-next';
  import nearestColor from 'nearest-color';
  import { colornames } from 'color-name-list';
  import Navbar from './components/Navbar.vue';
  import CopyCode from './components/CopyCode.vue';
  import Authentication from './components/Authentication.vue';

  // #region Primary Color Palette

  const colorPalette = ref<any[]>([]);
  const colorName = ref<string>('');
  const colorPickerRef = ref<InstanceType<typeof ColorPicker> | null>(null);

  const onColorChange = (color: string) => {
    const root = document.documentElement;

    const newPalette = Object.entries(
      tailwindcssPaletteGenerator({
        colors: [color],
        names: ['theme'],
      }).theme
    );

    colorPalette.value = newPalette?.map((item: string[]) => {
      const variableName = `--twc-theme-${item[0]}`,
        color = item[1];

      root.style.setProperty(variableName, color);

      return {
        name: variableName,
        level: item[0],
        color: color,
      };
    });

    const colors = colornames.reduce(
      (o: any, { name, hex }: any) => Object.assign(o, { [name]: hex }),
      {}
    );

    const nearest = nearestColor.from(colors);

    colorName.value = nearest(color);
  };

  // #endregion

  // #region Secondary Color Palette

  const secondaryColorPalette = ref<any[] | null>(null);
  const secondaryColorName = ref<string>('');
  const secondaryColorPickerRef = ref<InstanceType<typeof ColorPicker> | null>(
    null
  );

  const onSecondaryColorChange = (color: string) => {
    const root = document.documentElement;

    const newPalette = Object.entries(
      tailwindcssPaletteGenerator({
        colors: [color],
        names: ['theme'],
      }).theme
    );

    secondaryColorPalette.value = newPalette?.map((item: string[]) => {
      const variableName = `--twc-theme2-${item[0]}`,
        color = item[1];

      root.style.setProperty(variableName, color);

      return {
        name: variableName,
        level: item[0],
        color: color,
      };
    });

    const colors = colornames.reduce(
      (o: any, { name, hex }: any) => Object.assign(o, { [name]: hex }),
      {}
    );

    const nearest = nearestColor.from(colors);

    secondaryColorName.value = nearest(color);
  };

  const addSecondaryColor = () => {
    secondaryColorPalette.value = [];
    secondaryColorPickerRef.value?.generateRandomColor();
  };

  // #endregion

  // #region Random Color Generation

  const spacebarPress = ref<boolean>(false);

  const generateRandomColors = () => {
    colorPickerRef.value?.generateRandomColor();
    secondaryColorPickerRef.value?.generateRandomColor();
  };

  const handleSpacePress = (event: KeyboardEvent) => {
    if (event.code === 'Space') {
      generateRandomColors();

      spacebarPress.value = true;

      setTimeout(() => {
        spacebarPress.value = false;
      }, 250);
    }
  };

  // #endregion

  onMounted(() => {
    window.addEventListener('keydown', handleSpacePress);
  });

  onUnmounted(() => {
    window.removeEventListener('keydown', handleSpacePress);
  });
</script>
