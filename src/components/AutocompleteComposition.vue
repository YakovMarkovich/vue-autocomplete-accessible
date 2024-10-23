<template>
  <div ref="container">
    <label for="cb1-input">{{ label }}</label>
    <div class="combobox combobox-list">
      <div class="group">
        <input
          type="text"
          role="combobox"
          aria-autocomplete="both"
          aria-controls="autocomplete-list"
          :aria-expanded="isShow"
          :aria-activedescendant="
            activeOptionIndex >= 0 ? filteredOptions[activeOptionIndex]?.id : ''
          "
          ref="input"
          @input="handleInput"
          @keydown="handleKeyDown"
          @focus="handleFocus"
        />
        <button
          @click="handleToggleDropDown"
          aria-label="Toggle dropdown"
          :aria-expanded="isShow"
          tabIndex="-1"
        >
          <svg
            width="18"
            height="16"
            aria-hidden="true"
            focusable="false"
            forced-color-adjust:
            auto
          >
            <polygon
              class="arrow"
              stroke-width="0"
              fill-opacity="0.75"
              fill="currentcolor"
              points="3,6 15,6 9,14"
            ></polygon>
          </svg>
        </button>
      </div>
      <ul
        id="autocomplete-list"
        role="listbox"
        :aria-label="label"
        v-show="isShow"
        ref="listbox"
      >
        <li
          v-for="(item, index) in filteredOptions"
          :key="item.id"
          @click="handleOptionClick(item)"
          role="option"
          :id="item.id"
          :value="item.value"
          :class="index === activeOptionIndex ? 'active' : ''"
          :aria-selected="index === activeOptionIndex"
        >
          {{ item.value }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import {
  onBeforeUnmount,
  onMounted,
  ref,
  watch,
  defineProps,
  useTemplateRef,
} from "vue";

const props = defineProps({
  label: {
    type: String,
    required: false,
    default: "Default label",
  },
  items: {
    type: Array,
    required: true,
  },
});

const inputValue = ref("");
const currentOption = ref("");
const filteredOptions = ref([]);
const isShow = ref(false);
const activeOptionIndex = ref(-1);
const input = useTemplateRef("input");
const listbox = useTemplateRef("listbox");
const container = useTemplateRef("container");

const handleInput = (event) => {
  const value = event.target.value;
  inputValue.value = value;
  filteredOptions.value = props.items.filter((item) =>
    item.value.toLowerCase().startsWith(inputValue.value.toLowerCase())
  );
  if (filteredOptions.value.length) {
    isShow.value = true;
  }
  activeOptionIndex.value = -1;
  if (filteredOptions.value.length > 0) {
    if (currentOption.value.includes(value)) {
      currentOption.value = filteredOptions.value[0].value;
      input.value.value = value;
      return;
    }

    const suggestion = filteredOptions.value[0];
    currentOption.value = suggestion.value;
    input.value.value = suggestion.value;
  } else {
    currentOption.value = value;
    input.value.value = value;
  }
};

const handleOptionClick = (item) => {
  console.log("handleOptionClick");

  currentOption.value = item.value;
  inputValue.value = currentOption.value;
  input.value.value = currentOption.value;
  isShow.value = false;
};

const handleToggleDropDown = () => {
  if (!isShow.value) {
    filteredOptions.value = props.items;
    isShow.value = true;
    input.value.focus();
  } else {
    isShow.value = false;
  }
};

const handleKeyDown = (event) => {
  if (event.key === "ArrowDown") {
    const newIndex =
      (activeOptionIndex.value + 1) % filteredOptions.value.length;
    currentOption.value = filteredOptions.value[newIndex].value;
    inputValue.value = currentOption.value;
    activeOptionIndex.value = newIndex;
    input.value.value = filteredOptions.value[newIndex].value;
    setCurrentOptionStyle(filteredOptions.value[newIndex].value);
  } else if (event.key === "ArrowUp") {
    const newIndex =
      (activeOptionIndex.value - 1 + filteredOptions.value.length) %
      filteredOptions.value.length;
    currentOption.value = filteredOptions.value[newIndex].value;
    inputValue.value = currentOption.value;
    input.value.value = filteredOptions.value[newIndex].value;
    activeOptionIndex.value = newIndex;
    setCurrentOptionStyle(filteredOptions.value[newIndex].value);
    setTimeout(() => {
      input.value.setSelectionRange(
        input.value.value.length,
        input.value.value.length
      );
    }, 0);
  } else if (event.key === "Enter" && activeOptionIndex.value >= 0) {
    handleOptionClick(filteredOptions.value[activeOptionIndex.value]);
  } else if (event.key === "Escape") {
    if (isShow.value) {
      isShow.value = false;
    } else {
      currentOption.value = "";
      inputValue.value = "";
      input.value.value = "";
    }
  }
};

const setCurrentOptionStyle = (option) => {
  filteredOptions.value.forEach((opt) => {
    const optAsNode = document.getElementById(opt.id);
    if (opt.value === option) {
      if (
        listbox.value.scrollTop + listbox.value.offsetHeight <
        optAsNode.offsetTop + optAsNode.offsetHeight
      ) {
        listbox.value.scrollTop =
          optAsNode.offsetTop +
          optAsNode.offsetHeight -
          listbox.value.offsetHeight;
      } else if (listbox.value.scrollTop > optAsNode.offsetTop + 2) {
        listbox.value.scrollTop = opt.offsetTop;
      }
    }
  });
};

const handleClickOutside = (event) => {
  if (container.value && !container.value.contains(event.target)) {
    isShow.value = false;
  }
};

const handleFocus = () => {
  if (currentOption.value === "") {
    filteredOptions.value = props.items;
    isShow.value = true;
  }
};

const setSelection = () => {
  input.value.setSelectionRange(
    inputValue.value.length,
    currentOption.value.length
  );
};
watch(inputValue, () => {
  setSelection();
});
watch(currentOption, () => {
  setSelection();
});

onMounted(() => {
  document.addEventListener("mousedown", handleClickOutside);
});

onBeforeUnmount(() => {
  document.removeEventListener("mousedown", handleClickOutside);
});
</script>

<style src="../assets/AutocompleteAccessible.css"></style>
