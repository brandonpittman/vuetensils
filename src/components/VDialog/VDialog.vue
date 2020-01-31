<template>
  <div
    v-if="isVisible"
    :class="classes.root"
    @click="onClick"
    @keydown="onKeydown"
  >
    <component
      :is="tag"
      ref="content"
      :class="classes.content"
      tabindex="-1"
      role="dialog"
    >
      <slot />
    </component>
  </div>
</template>

<script>
import KEYCODES from "../../data/keycodes"
import FOCUSABLE from "../../data/focusable"

export default {
  model: {
    prop: "isVisible",
    event: "change",
  },
  props: {
    isVisible: Boolean,
    tag: {
      type: String,
      default: "div",
    },
    dismissible: {
      type: Boolean,
      default: true,
    },
    noScroll: {
      type: Boolean,
      default: false,
    },
    classes: {
      type: Object,
      default: () => ({}),
    },
  },
  watch: {
    isVisible: {
      handler(next, prev) {
        if (typeof window !== "undefined") {
          if (next && next != prev) {
            this.noScroll &&
              document.body.style.setProperty("overflow", "hidden")
            this.$nextTick(() => {
              this.$refs.content.focus()
            })
          } else {
            this.noScroll && document.body.style.removeProperty("overflow")
          }
        }
      },
    },
  },

  methods: {
    show() {
      this.$emit("show")
      this.$emit("change", true)
    },
    hide() {
      this.$emit("hide")
      this.$emit("change", false)
    },
    toggle() {
      const { isVisible } = this
      const event = isVisible ? "hide" : "show"
      this.$emit(event, !isVisible)
      this.$emit("change", !isVisible)
    },
    onClick(event) {
      if (event.target.classList.contains("vts-dialog") && this.dismissible) {
        this.hide()
      }
    },

    onKeydown(event) {
      if (event.keyCode === KEYCODES.ESC) {
        this.hide()
      }
      if (event.keyCode === KEYCODES.TAB) {
        const content = this.$refs.content
        if (!content) return

        const focusable = Array.from(content.querySelectorAll(FOCUSABLE))

        if (!focusable.length) {
          event.preventDefault()
          return
        }

        if (!content.contains(document.activeElement)) {
          event.preventDefault()
          focusable[0].focus()
        } else {
          const focusedItemIndex = focusable.indexOf(document.activeElement)

          if (event.shiftKey && focusedItemIndex === 0) {
            focusable[focusable.length - 1].focus()
            event.preventDefault()
          }

          if (!event.shiftKey && focusedItemIndex === focusable.length - 1) {
            focusable[0].focus()
            event.preventDefault()
          }
        }
      }
    },
  },
}
</script>
