<template>
  <div v-if="show" class="layout-column">
    <div class="m-4 py-1 px-2 bg-gray-900 rounded">
      {{ t("item.identification", [baseType]) }}
    </div>
    <div class="grid grid-cols-2 gap-2 overflow-auto pb-4 px-4">
      <div v-for="item in identifiedVariants" :key="item.name" class="flex">
        <button
          @click="select(item)"
          class="bg-gray-700 rounded flex gap-x-3 items-center p-2 w-full"
        >
          <img
            :src="
              item.icon === '%NOT_FOUND%' || item.icon === ''
                ? '/images/404.png'
                : item.icon
            "
            class="w-12"
          />
          <div class="leading-tight text-left">{{ item.name }}</div>
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, computed } from "vue";
import { useI18n } from "vue-i18n";
import { BaseType, ITEMS_ITERATOR } from "@/assets/data";
import { ItemRarity, ParsedItem } from "@/parser";

export default defineComponent({
  emits: ["identify"],
  props: {
    item: {
      type: Object as PropType<ParsedItem | null>,
      default: null,
    },
  },
  setup(props, ctx) {
    const identifiedVariants = computed(() => {
      const baseType = props.item!.info.refName;
      const possible: BaseType[] = [];
      for (const match of ITEMS_ITERATOR(JSON.stringify(baseType))) {
        if (match.namespace === "UNIQUE" && match.unique!.base === baseType) {
          // TODO currently ignoring variants
          if (!possible.some((unique) => unique.refName === match.refName)) {
            possible.push(match);
          }
        }
      }
      if (possible.length === 1) {
        select(possible[0]);
      }

      return possible;
    });

    const show = computed(() => {
      if (!props.item) return false;

      return (
        props.item.rarity === ItemRarity.Unique &&
        props.item.isUnidentified &&
        !props.item.info.unique
      );
    });

    function select(info: BaseType) {
      const newItem: ParsedItem = {
        ...props.item!,
        info,
      };
      ctx.emit("identify", newItem);
    }

    const { t } = useI18n();

    return {
      t,
      identifiedVariants,
      show,
      baseType: computed(() => props.item!.info.name),
      select,
    };
  },
});
</script>
