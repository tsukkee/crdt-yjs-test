<template>
  <div class="info">
    <div>Network status: {{ networkStatus }}</div>
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
    <div>
      <button @click="undo">←undo</button> <button @click="redo">→redo</button>
    </div>
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
      <input v-model="inputText" type="text" /><button @click="addItem">
        add
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, customRef } from "vue";
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

// prettier-ignore
const yArrayRef = <T,>(yarray: Y.Array<T>) => {
  return customRef<T[]>((track, trigger) => {
    yarray.observe(() => {
      trigger();
    });

    return {
      get() {
        track();
        return yarray.toArray();
      },
      set(newValue: T[]) {
        yarray.doc?.transact(() => {
          yarray.delete(0, yarray.length);
          yarray.insert(0, newValue);
          trigger();
        });
      },
    };
  });
};

export default defineComponent({
  components: {
    ListItem,
  },
  setup() {
    // yjs
    const doc = new Y.Doc();
    const wsProvider = new WebsocketProvider(
      "ws://localhost:12345",
      "my-list",
      doc
    );

    // network status
    const networkStatus = ref("disconnected");
    wsProvider.on("status", (event: { status: string }) => {
      networkStatus.value = event.status;
    });

    // awareness
    const users = ref<User[]>([]);
    const userMap = computed(() => {
      const map = new Map<number, string>();
      users.value.forEach((user) => {
        map.set(user.index, user.name);
      });
      return map;
    });

    const awareness = wsProvider.awareness;
    awareness.on("change", () => {
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

    const userName = ref("");
    const focusedIndex = ref(-1);
    const setUserName = () => {
      awareness.setLocalStateField("user", {
        name: userName.value,
        index: focusedIndex.value,
      });
    };
    const focusItem = (index: number) => {
      focusedIndex.value = index;
      setUserName();
    };

    const blurItem = () => {
      focusedIndex.value = -1;
      setUserName();
    };

    // list data
    const yarray = doc.getArray<string>("my-array");
    const list = yArrayRef(yarray);

    const inputText = ref("");
    const addItem = () => {
      yarray.insert(list.value.length, [inputText.value]);
      inputText.value = "";
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

    // undo/redo
    const undoManager = new Y.UndoManager(yarray);
    const undo = () => undoManager.undo();
    const redo = () => undoManager.redo();

    return {
      // network status
      networkStatus,

      // awareness
      userName,
      users,
      userMap,
      setUserName,
      focusItem,
      blurItem,

      // list data
      list,
      inputText,
      addItem,
      updateItem,
      removeItem,

      // undo/redo
      undo,
      redo,
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
