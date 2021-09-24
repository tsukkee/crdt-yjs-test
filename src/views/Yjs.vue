<template>
  <div class="info">
    <div>
      User name: <input v-model="userName" type="text" /><button
        @click="setUserName"
      >
        set
      </button>
    </div>
    Users:
    <ul>
      <li v-for="user in users" :key="user.id">{{ user.name }}</li>
    </ul>
  </div>
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

type User = {
  id: number;
  name: string;
};

type UserState = {
  name: string;
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

      users.value = Array.from(awareness.getStates().entries()).map(
        ([key, value]) => {
          const user = value.user as UserState;
          return {
            id: key,
            name: user.name,
          };
        }
      );
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

    const setUserName = () => {
      awareness.setLocalStateField("user", {
        name: userName.value,
      });
    };

    const userName = ref("");
    const inputText = ref("");
    const list = ref<string[]>([]);
    const users = ref<User[]>([]);

    return {
      addItem,
      updateItem,
      removeItem,
      setUserName,
      userName,
      inputText,
      list,
      users,
    };
  },
});
</script>

<style lang="scss" scoped>
.info {
  margin-bottom: 12px;

  li {
    width: 20em;
    text-align: left;
  }
}

.list {
  display: flex;
  flex-direction: column;
  gap: 4px;
  align-items: center;
}
</style>
