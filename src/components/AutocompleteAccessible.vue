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

<script>
export default {
  name: "AutocompleteAccessible",
  props: {
    label: {
      type: String,
      required: false,
      default: "Default label",
    },
    items: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      inputValue: "",
      currentOption: "",
      filteredOptions: [],
      isShow: false,
      activeOptionIndex: -1,
    };
  },
  methods: {
    handleInput(event) {
      const value = event.target.value;
      this.inputValue = value;
      this.filteredOptions = this.items.filter((item) =>
        item.value.toLowerCase().startsWith(this.inputValue.toLowerCase())
      );
      if (this.filteredOptions.length) {
        this.isShow = true;
      }
      this.activeOptionIndex = -1;
      if (this.filteredOptions.length > 0) {
        console.log("in > 0");
        if (this.currentOption.includes(value)) {
          this.currentOption = this.filteredOptions[0].value;
          this.$refs.input.value = value;
          //this.$refs.input.value = this.currentOption;
          return;
        }
      
        const suggestion = this.filteredOptions[0].value;
        this.currentOption = suggestion;
        this.$refs.input.value = suggestion;
      } else {
        this.currentOption = value;
        this.$refs.input.value = value;
      }
    },
    handleOptionClick(item) {
      this.currentOption = item.value;
      this.inputValue = this.currentOption;
      this.$refs.input.value = this.currentOption;
      this.isShow = false;
    },
    handleToggleDropDown() {
      if (!this.isShow) {
        this.filteredOptions = this.items;
        this.isShow = true;
        this.$refs.input.focus();
      } else {
        this.isShow = false;
      }
    },
    handleKeyDown(event) {
      if (event.key === "ArrowDown") {
        const newIndex =
          (this.activeOptionIndex + 1) % this.filteredOptions.length;
        this.currentOption = this.filteredOptions[newIndex].value;
        this.inputValue = this.currentOption;
        this.activeOptionIndex = newIndex;
        this.$refs.input.value = this.filteredOptions[newIndex].value;
        this.setCurrentOptionStyle(this.filteredOptions[newIndex].value);
      } else if (event.key === "ArrowUp") {
        const newIndex =
          (this.activeOptionIndex - 1 + this.filteredOptions.length) %
          this.filteredOptions.length;
        this.currentOption = this.filteredOptions[newIndex].value;
        this.inputValue = this.currentOption;
        this.$refs.input.value = this.filteredOptions[newIndex].value;
        this.activeOptionIndex = newIndex;
        this.setCurrentOptionStyle(this.filteredOptions[newIndex].value);
        setTimeout(() => {
          this.$refs.input.setSelectionRange(
            this.$refs.input.value.length,
            this.$refs.input.value.length
          );
        }, 0);
      } else if (event.key === "Enter" && this.activeOptionIndex >= 0) {
        this.handleOptionClick(this.filteredOptions[this.activeOptionIndex]);
      } else if (event.key === "Escape") {
        if (this.isShow) {
          this.isShow = false;
        } else {
          this.currentOption = "";
          this.inputValue = "";
          this.$refs.input.value = "";
        }
      }
    },
    setCurrentOptionStyle(option) {
      this.filteredOptions.forEach((opt) => {
        const optAsNode = document.getElementById(opt.id);
        if (opt.value === option) {
          if (
            this.$refs.listbox.scrollTop + this.$refs.listbox.offsetHeight <
            optAsNode.offsetTop + optAsNode.offsetHeight
          ) {
            this.$refs.listbox.scrollTop =
              optAsNode.offsetTop +
              optAsNode.offsetHeight -
              this.$refs.listbox.offsetHeight;
          } else if (this.$refs.listbox.scrollTop > optAsNode.offsetTop + 2) {
            this.$refs.listbox.scrollTop = opt.offsetTop;
          }
        }
      });
    },
    handleClickOutside(event) {
      if (
        this.$refs.container &&
        !this.$refs.container.contains(event.target)
      ) {
        this.isShow = false;
      }
    },
    handleFocus() {
      if (this.currentOption === "") {
        this.filteredOptions = this.items;
        this.isShow = true;
      }
    },
    setSelection() {
      console.log("input value ", this.inputValue);
      console.log("current option value ", this.currentOption);
      console.log("current input ref value ", this.$refs.input.value);
      //this.$refs.input.value = this.currentOption;
      this.$refs.input.setSelectionRange(
        this.inputValue.length,
        this.currentOption.length
      );
    },
  },
  watch: {
    inputValue() {
      this.setSelection();
    },
    currentOption() {
      this.setSelection();
    },
  },
  mounted() {
    document.addEventListener("mousedown", this.handleClickOutside);
  },
  beforeUnmount() {
    document.removeEventListener("mousedown", this.handleClickOutside);
  },
};
</script>

<style src="../assets/AutocompleteAccessible.css"></style>
