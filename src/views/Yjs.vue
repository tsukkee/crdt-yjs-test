<template>
  <div class="info">
    <button @click="undo">←undo</button> <button @click="redo">→redo</button>
    <div>
      User name: <input v-model="userName" type="text" /><button
        @click="setUserName"
      >
        set
      </button>
    </div>
    Other users:
    <ul>
      <li v-for="user in users" :key="user.id">
        {{ user.name }}
      </li>
    </ul>
  </div>
  <div class="list">
    <ListItem
      v-for="(item, index) in list"
      :key="item + '_' + index"
      :text="item"
      :index="index"
      :user="userMap.get(index)"
      @update="updateItem"
      @remove="removeItem"
      @focus="focusItem"
      @blur="blurItem"
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
import { defineComponent, ref, computed } from "vue";
import * as Y from "yjs";
import { WebsocketProvider } from "y-websocket";
import ListItem from "../components/ListItem.vue";

type User = {
  id: number;
  name: string;
  index: number;
};

type UserState = {
  name: string;
  index: number;
};

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
    const awareness = wsProvider.awareness;
    awareness.on("change", (changes: any[]) => {
      console.log("awareness", changes, awareness.getStates());

      users.value = Array.from(awareness.getStates().entries())
        .map(([key, value]) => {
          const user = value.user as UserState | undefined;
          return {
            id: key,
            name: user?.name ?? "",
            index: user?.index ?? -1,
          };
        })
        .filter((user) => user.id !== doc.clientID);
    });

    const yarray = doc.getArray<string>("my-array");
    yarray.observe(() => {
      list.value = yarray.toArray();
    });

    const undoManager = new Y.UndoManager(yarray);

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

    const focusItem = (index: number) => {
      focusedIndex.value = index;
      setUserName();
    };

    const blurItem = () => {
      focusedIndex.value = -1;
      setUserName();
    };

    const setUserName = () => {
      awareness.setLocalStateField("user", {
        name: userName.value,
        index: focusedIndex.value,
      });
    };

    const undo = () => undoManager.undo();
    const redo = () => undoManager.redo();

    const userName = ref("");
    const inputText = ref("");
    const list = ref<string[]>([]);
    const users = ref<User[]>([]);
    const focusedIndex = ref(-1);

    const userMap = computed(() => {
      const map = new Map<number, string>();
      users.value.forEach((user) => {
        map.set(user.index, user.name);
      });
      return map;
    });

    return {
      addItem,
      updateItem,
      removeItem,
      focusItem,
      blurItem,
      setUserName,
      undo,
      redo,
      userName,
      userMap,
      inputText,
      list,
      users,
    };
  },
});
</script>

<style lang="scss" scoped>
.info {
  width: 25em;
  margin: 0 auto 12px auto;
  text-align: left;

  ul {
    margin: 0;
  }
}

.list {
  display: flex;
  flex-direction: column;
  gap: 4px;
  align-items: center;
}
</style>
