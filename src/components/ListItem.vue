<template>
  <div class="list-item">
    <template v-if="isEditing">
      <input type="text" v-model="inputText" /><button @click="finishEditing">
        done
      </button>
    </template>
    <template v-else>
      <span>{{ inputText }} </span><button @click="startEditing">edit</button>
      <button @click="remove">remove</button>
    </template>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
  props: {
    text: {
      type: String,
      required: true,
    },
    index: {
      type: Number,
      required: true,
    },
  },
  setup(props, context) {
    const isEditing = ref(false);

    const inputText = ref(props.text);

    const startEditing = () => (isEditing.value = true);
    const finishEditing = () => {
      isEditing.value = false;
      context.emit("update", inputText.value, props.index);
    };
    const remove = () => {
      context.emit("remove", props.index);
    };

    return {
      inputText,
      startEditing,
      finishEditing,
      remove,
      isEditing,
    };
  },
});
</script>

<style lang="scss" scoped>
.list-item {
  display: flex;
  gap: 4px;
  width: 25em;
  background: #eee;
  border-radius: 4px;
  padding: 4px;

  span,
  input {
    text-align: left;
    flex-grow: 1;
  }
  button {
    flex-grow: 0;
  }
}
</style>
