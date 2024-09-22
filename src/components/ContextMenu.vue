<script setup lang="ts" generic="TItem">
import { nextTick, onBeforeUpdate, ref } from 'vue';

interface ContextMenuProps {
  items: TItem[];
  menuBgColor?: string;
  optionHighlightColor?: string;
  optionActiveColor?: string;
  optionFocusColor?: string;
  selectedBgColor?: string;
  isMenuVisible?: boolean;
}

const props = withDefaults(defineProps<ContextMenuProps>(), {
  menuBgColor: 'var(--tertery-clr)',
  optionFocusColor: 'rgba(64, 64, 64, 0.87)',
  optionHighlightColor: 'var(--accent-light-clr)',
  optionActiveColor: 'hsl(0, 0%, 47%)',
  selectedBgColor: 'hsl(0, 0%, 35%)'
});

const emit = defineEmits<{
  (evt: 'close', isOpen: boolean): void;
}>();

const model = defineModel();
model.value = props.items[0];

const currentItemIndex = ref<number>(0);
const allMenuItems = ref<HTMLLIElement[]>([]);
const collectMenuItems = (el: HTMLLIElement) => allMenuItems.value.push(el);

const selectItem = (option: TItem) => {
  model.value = option;
  emit('close', false);
};

function highlightPrevOption(evt: KeyboardEvent) {
  if (currentItemIndex.value === 0) {
    currentItemIndex.value = props.items.length - 1;
    onKeydown(evt, currentItemIndex.value);
  } else {
    currentItemIndex.value -= 1;
    onKeydown(evt, currentItemIndex.value);
  }
  allMenuItems.value[currentItemIndex.value].focus();
}

async function highlightNextItem(evt: KeyboardEvent) {
  if (currentItemIndex.value >= props.items.length - 1) {
    currentItemIndex.value = 0;
  } else {
    currentItemIndex.value += 1;
  }
  allMenuItems.value[currentItemIndex.value].focus();
}

function selectAndFocusOption() {
  const option = props.items[currentItemIndex.value];
  selectItem(option as TItem);
  setItemFocus(option as Element);
}

function handleKeyDown(evt: KeyboardEvent) {
  switch (evt.key) {
    case 'Escape':
      emit('close', false);
      break;
    case 'ArrowUp':
      highlightPrevOption(evt);
      break;
    case 'ArrowDown':
      highlightNextItem(evt);
      break;
    case 'Enter':
      selectAndFocusOption();
      break;
  }
}

async function setItemFocus(listEl: Element | object) {
  const option = listEl as HTMLLIElement;
  collectMenuItems(option);
  await nextTick();
  if (props.isMenuVisible) {
    if (option.textContent === model.value) {
      option.focus();
    }
  } else {
    currentItemIndex.value = props.items.findIndex(
      (opt) => opt === model.value
    );
  }
}

onBeforeUpdate(() => {
  allMenuItems.value = [];
});

const onKeydown = async (evt: KeyboardEvent, index?: number) => {
  if (evt.key !== 'ArrowDown' && evt.key !== 'ArrowUp') {
    return;
  }
  const items = allMenuItems.value;
  if (evt.key === 'ArrowDown') {
    items[index!].focus();
  } else if ('ArrowUp') {
    items[0].focus();
  }
};
</script>

<template>
  <Transition name="unravel" mode="out-in" appear>
    <ul v-if="isMenuVisible" id="ctx-menu" class="context-menu" role="menu">
      <li
        v-for="(option, index) in items"
        :key="(option as keyof TItem)"
        :ref="(el) => setItemFocus(el!)"
        @click="selectItem(option)"
        @keydown="handleKeyDown"
        :class="{ 'is-selected': option === model }"
        class="context-menu-item"
        tabindex="0"
        role="menuitem"
        :aria-label="`Option ${index + 1}: ${option}`"
        :aria-selected="option === model"
      >
        <span>{{ option }}</span>
      </li>
    </ul>
  </Transition>
</template>

<style scoped>
.context-menu {
  border-radius: 4px;
  background-color: v-bind(menuBgColor);
  overflow: hidden;
  cursor: pointer;
}

.context-menu li {
  padding: 0.23rem 1rem;
  font-size: 0.875rem;
}

.context-menu-item:focus-visible,
.context-menu-item:focus {
  border: none;
  outline: none;
  background-color: v-bind(optionFocusColor);
}

.context-menu-item.is-selected {
  background-color: v-bind(selectedBgColor);
}

.context-menu-item:hover {
  background-color: v-bind(optionHighlightColor);
}

.context-menu-item:active {
  background-color: v-bind(optionActiveColor);
}
</style>
