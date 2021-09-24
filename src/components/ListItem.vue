<template>
  <div class="list-item">
    <template v-if="isEditing">
      <input
        type="text"
        v-model="inputText"
        @focus="focus"
        @blur="blur"
      /><button @click="finishEditing">done</button>
    </template>
    <template v-else>
      <span>{{ inputText }} </span><button @click="startEditing">edit</button>
      <button @click="remove">remove</button>
    </template>
    <div v-if="user" class="user">{{ user }} is editing</div>
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
    user: {
      type: String,
      default: null,
    },
  },
  setup(props, context) {
    const isEditing = ref(false);

    const inputText = ref(props.text);

    const startEditing = () => (isEditing.value = true);
    const finishEditing = () => {
      isEditing.value = false;
      blur();
      context.emit("update", inputText.value, props.index);
    };
    const remove = () => {
      context.emit("remove", props.index);
    };

    const focus = () => context.emit("focus", props.index);
    const blur = () => context.emit("blur", props.index);

    return {
      inputText,
      startEditing,
      finishEditing,
      remove,
      focus,
      blur,
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

  position: relative;

  span,
  input {
    text-align: left;
    flex-grow: 1;
  }
  button {
    flex-grow: 0;
  }

  .user {
    position: absolute;
    left: calc(25em + 8px);
    background: #fff;
    color: #888;
    border: 1px solid #eee;
    width: max-content;
  }
}
</style>
