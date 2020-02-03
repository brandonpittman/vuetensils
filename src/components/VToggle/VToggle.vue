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

    <div class="vts-dialog__wrapper">
      <transition name="vts-toggle-fade">
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
}
</script>

<style>
.vts-toggle-fade-enter-active {
  transition: all 300ms ease-in-out;
  transform-origin: top;
}

.vts-toggle-fade-leave-active {
  transition: all 300ms ease-in-out;
  transform-origin: top;
}

.vts-toggle-fade-enter,
.vts-toggle-fade-leave-to {
  opacity: 0;
  transform: translateY(-100%);
}

.vts-dialog__wrapper {
  overflow: hidden;
}
</style>
