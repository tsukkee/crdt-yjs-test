<template>
  <div class="list">
    <ListItem
      v-for="(item, index) in list"
      :key="item + '_' + index"
      :text="item"
      :index="index"
      @update="updateItem"
      @remove="removeItem"
    />
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
import { WebsocketProvider } from "y-websocket";
import ListItem from "../components/ListItem.vue";

export default defineComponent({
  components: {
    ListItem,
  },
  setup() {
    const doc = new Y.Doc();
    const wsProvider = new WebsocketProvider(
      "ws://localhost:12345",
      "my-list",
      doc
    );
    wsProvider.on("status", (event: Event) => {
      console.log("WebsocketProvider status", event);
    });

    const yarray = doc.getArray<string>("my-array");
    yarray.observe(() => {
      list.value = yarray.toArray();
    });

    const addItem = (text: string) => {
      yarray.insert(list.value.length, [text]);
    };

    const updateItem = (text: string, index: number) => {
      doc.transact(() => {
        yarray.delete(index);
        yarray.insert(index, [text]);
      });
    };

    const removeItem = (index: number) => {
      yarray.delete(index);
    };

    const inputText = ref("");
    const list = ref<string[]>([]);

    return {
      addItem,
      updateItem,
      removeItem,
      inputText,
      list,
    };
  },
});
</script>

<style lang="scss" scoped>
.list {
  display: flex;
  flex-direction: column;
  gap: 4px;
  align-items: center;
}
</style>
