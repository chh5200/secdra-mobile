<template>
  <label :class="classObject" class="radio">
    <slot>
      <div
        v-ripple="!disabled && { class: `${color}-text` }"
        :class="{ 'dark-fade-text': !model }"
        class="radio-inner"
      >
        <div class="content">
          <i
            :class="{ 'ali-icon-radio': !model, 'ali-icon-radio-fill': model }"
            class="icon"
          ></i>
        </div>
      </div>
    </slot>
    <span
      v-if="label !== null"
      :class="{ 'dark-text': !model }"
      class="radio-label"
      >{{ label }}</span
    >
    <input
      v-model="model"
      :disabled="disabled"
      :value="true"
      type="radio"
      class="radio-original"
    />
  </label>
</template>

<script>
export default {
  componentName: "Radio",
  model: {
    prop: "input",
    event: "input"
  },
  props: {
    input: {
      type: String | Number | Object | Boolean,
      default: null
    },
    label: {
      type: String | Number,
      default: null
    },
    disabled: {
      type: Boolean,
      default: false
    },
    color: {
      type: String,
      default: "default"
    },
    big: {
      type: Boolean,
      default: false
    },
    small: {
      type: Boolean,
      default: false
    },
    // 如果value是Object的话valueKey必填
    value: {
      type: String | Number | Object | Boolean,
      default: true
    },
    valueKey: {
      type: String | Number,
      default: null
    }
  },
  data() {
    return {
      radioGroup: {},
      selfModel: this.input
    }
  },
  computed: {
    model: {
      get() {
        if (this.isGroup) {
          if (typeof this.value === "object") {
            return this.radioGroup.exist(this.value, this.valueKey)
          } else {
            return this.radioGroup.exist(this.value)
          }
        } else if (typeof this.value === "object") {
          return this.input[this.valueKey] === this.value[this.valueKey]
        } else {
          return this.input === this.value
        }
      },
      set() {
        if (this.isGroup) {
          this.radioGroup.change(this.value)
        } else {
          this.$emit("input", this.value)
        }
      }
    },
    isGroup() {
      let parent = this.$parent
      while (parent) {
        if (parent.$options.componentName !== "RadioGroup") {
          parent = parent.$parent
        } else {
          this.radioGroup = parent
          return true
        }
      }
      return false
    },
    classObject() {
      const classObject = {}
      classObject[this.color + "-color"] = true
      classObject.small = this.small
      classObject.big = this.big
      classObject.disabled = this.disabled
      return classObject
    }
  },
  watch: {
    input(newVal) {
      this.selfModel = newVal
    }
  }
}
</script>

<style type="text/less" lang="less" scoped></style>
