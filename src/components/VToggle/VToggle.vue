<template>
  <div :class="styling.root">
    <button
      :id="`${id}-label`"
      ref="label"
      :disabled="disabled"
      :aria-controls="`${id}-content`"
      :aria-expanded="isOpen"
      :class="styling.label"
      @click="isOpen = !isOpen"
    >
      <!-- @slot The content that goes inside the button -->
      <slot name="label" />
    </button>

    <transition name="slide-fade">
      <div
        v-show="isOpen && !disabled"
        :id="`${id}-content`"
        ref="content"
        :aria-labelledby="`${id}-label`"
        :aria-hidden="!isOpen"
        role="region"
        :class="styling.content"
      >
        <!-- @slot The content that goes inside the toggleable region -->
        <slot />
      </div>
    </transition>
  </div>
</template>

<script>
/**
 * Toggle the visibility of content. Useful for something like an FAQ page, for example. Includes ARIA attributes for expandable content and is keyboard friendly.
 */
export default {
  props: {
    /**
     * The content inside the toggle button
     */
    label: {
      type: String,
      required: true,
    },

    disabled: Boolean,

    utilities: {
      type: Object,
      default: undefined
    },
    classes: {
      type: Object,
      default: () => ({}),
    },
  },

  data() {
    return {
      isOpen: !!this.isOpen,
    }
  },

  computed: {
    styling() {
      if (this.utilities) return this.utilities

      return {
        root: ['vts-toggle', { 'vts-toggle--open': this.isOpen }, this.classes.root],
        label: ['vts-toggle__label', this.classes.label],
        content: ['vts-toggle__content', this.classes.content]
      }
    },
    id() {
      const { id } = this.$attrs
      if (id) return id

      return (
        "vts-toggle-" +
        Array(6)
          .fill()
          .map(() => Math.floor(36 * Math.random()).toString(36))
          .join("")
      )
    },
  },

  methods: {
    collapse(el) {
      this.$refs.content.style.transform = "scaleY(0)"
    },

    expand(el) {
      this.$refs.content.style.transform = "scaleY(1)"
      // el.style.overflow = "hidden"
      // el.style.height = `${el.scrollHeight}px`
      // Force repaint to make sure the animation is triggered correctly.
      // el.scrollHeight
    },

    resetHeight(el) {
      // el.style.overflow = "visible"
      // el.style.height = ""
    },
  },
}
</script>

<style>
.slide-fade-enter-active {
  position: relative;
  z-index: -1;
  transition: all 0.3s ease-in-out;
}

.slide-fade-leave-active {
  position: relative;
  z-index: -1;
  transition: all 0.8s ease-in-out;
}

.slide-fade-enter,
.slide-fade-leave-to {
  position: relative;
  z-index: -1;
  opacity: 0;
  transform: translateY(-100%);
}

.vts-toggle__content {
  transition: all 300ms ease-in-out;
}
</style>
