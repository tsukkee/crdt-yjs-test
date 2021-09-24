<template>
  <div>
    <ul>
      <li v-for="(item, index) in list" :key="index">
        {{ item }}
      </li>
    </ul>
    <div>
      <input v-model="inputText" type="text" /><button
        @click="addItem(inputText)"
      >
        add
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import * as Y from "yjs";

export default defineComponent({
  setup() {
    const doc = new Y.Doc();
    const yarray = doc.getArray<string>("my-array");

    yarray.observe(() => {
      list.value = yarray.toArray();
    });

    const addItem = (text: string) => {
      yarray.insert(list.value.length - 1, [text]);
    };

    const inputText = ref("");
    const list = ref<string[]>([]);

    return {
      addItem,
      inputText,
      list,
    };
  },
});
</script>

<style lang="scss" scoped></style>
