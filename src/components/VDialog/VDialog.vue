<template>
  <transition :name="transitionRoot">
    <div
      v-if="isVisible"
      :class="styling.root"
      @click="onClick"
      @keydown="onKeydown"
    >
      <transition
        :name="transitionContent"
        appear
      >
        <component
          :is="tag"
          ref="content"
          :class="styling.content"
          tabindex="-1"
          role="dialog"
        >
          <!-- @slot Content that exists within the dialog. -->
          <slot />
        </component>
      </transition>
    </div>
  </transition>
</template>

<script>
import KEYCODES from "../../data/keycodes"
import FOCUSABLE from "../../data/focusable"

/**
 * A dialog component for isVisible users content which overlays the rest of the applications. When opened, it traps the user's focus so that keyboard navigation will remain within the dialog until it is closed. It supports being closed by clicking outside the dialog content or pressing the ESC key.
 */
export default {
  model: {
    prop: "isVisible",
    event: "change",
  },

  props: {
    utilities: {
      type: Object,
      default: undefined
    },
    /**
     * @model
     */
    isVisible: Boolean,
    /**
     * HTML component for the dialog content.
     */
    tag: {
      type: String,
      default: "div",
    },
    /**
     * Flag to enable/prevent the dialog from being closed.
     */
    dismissible: {
      type: Boolean,
      default: true,
    },
    /**
    * Whether or not this bitch should scroll, yo.
    */
    noScroll: {
      type: Boolean,
      default: false,
    },
    /**
     * Transition name to apply to the dialog.
     */
    transitionContent: {
      type: String,
      default: ''
    },
    /**
     * Transition name to apply to the background.
     */
    transitionRoot: {
      type: String,
      default: ''
    },

    classes: {
      type: Object,
      default: () => ({}),
    },
  },
  computed: {
    styling() {
      if (this.utilities) return this.utilities

      return {
        root: ['vts-dialog', this.classes.root],
        content: ['vts-dialog__content', this.classes.content]
      }
    }
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
      /**
       * Fired when the dialog opens.
       * @event show
       * @type { boolean }
       */
      this.$emit("show")
      this.$emit("change", true)
    },
    hide() {
      /**
       * Fired when the dialog closes.
       * @event hide
       * @type { boolean }
       */
      this.$emit("hide")
      this.$emit("change", false)
    },
    toggle() {
      const { isVisible } = this
      const event = isVisible ? "hide" : "show"
      this.$emit(event, !isVisible)
      /**
       * Fired whenever the dialog opens or closes.
       * @event change
       * @type { boolean }
       */
      this.$emit("change", !isVisible)
    },
    onClick(event) {
      if ((event.target == this.$el) && this.dismissible) {
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

<style scoped>
.vts-dialog {
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  z-index: 100;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.4);
}

.vts-dialog [tabindex="-1"]:focus {
  outline: 0;
}

.fader-enter,
.fader-leave-to {
  opacity: 0;
}

.vts-dialog__content {
  overflow: auto;
  max-width: 70vw;
  max-height: 80vh;
  background: #fff;
  transition: all 0.5s ease-out;
}
</style>
