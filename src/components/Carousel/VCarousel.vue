
<script lang="ts">
import { computed, defineComponent, ref, watch } from 'vue-demi'

export default defineComponent({
  props: {
    component: {
      type: String,
      default: 'sa-carousel',
    },
    modelValue: {
      type: Number,
      default: 0,
    },
    vertical: {
      type: Boolean,
      default: false,
    },
    sliderClass: {
      type: String,
      default: '',
    },
  },
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    let scrollTimer: any
    const slider = ref<HTMLElement>()

    const nextSlide = (skip = 1) => {
      if (slider.value) {
        const newIndex = props.modelValue + skip
        emit('update:modelValue', newIndex)
      }
    }

    const prevSlide = (skip = 1) => {
      if (slider.value) {
        const newIndex = props.modelValue - skip
        emit('update:modelValue', newIndex)
      }
    }

    const maxPos = 5
    const timeoutInterval = 400
    const handleScroll = () => {
      clearInterval(scrollTimer)
      scrollTimer = setTimeout(() => {
        if (slider.value) {
          const sliderRect = slider.value.getBoundingClientRect()
          for (let index = 0; index < slider.value.children.length; index++) {
            const element = slider.value.children[index]
            if (element) {
              let relativePos
              if (props.vertical)
                relativePos = element.getBoundingClientRect().y - sliderRect.top
              else
                relativePos = element.getBoundingClientRect().x - sliderRect.left

              if (relativePos >= -maxPos && relativePos <= maxPos) {
                emit('update:modelValue', index)
                return
              }
            }
          }
          emit('update:modelValue', slider.value.children.length - 1)
        }
      }, timeoutInterval)
    }

    watch(
      () => props.modelValue,
      (val) => {
        if (slider.value) {
          let index = Math.min(val, slider.value.childElementCount - 1)
          index = Math.max(0, index)
          if (val === index) {
            const sliderRect = slider.value.getBoundingClientRect()
            const el = slider.value.children[val]
            if (props.vertical) {
              const scrollTotal = el.getBoundingClientRect().y - sliderRect.top
              slider.value.scrollTop += scrollTotal
            }
            else {
              const scrollTotal = el.getBoundingClientRect().x - sliderRect.left
              slider.value.scrollLeft += scrollTotal
            }
          } else {
            emit('update:modelValue', index)
          }
        }
      },
      { immediate: true },
    )

    const sliderDirection = computed(() => {
      return props.vertical ? 'vertical' : 'horizontal'
    })

    return {
      slider,
      nextSlide,
      prevSlide,
      handleScroll,
      sliderDirection,
    }
  },
})
</script>

<template>
  <div class="relative">
    <div
      ref="slider"
      :direction="sliderDirection"
      class="relative"
      :class="`${component} ${sliderClass}`"
      @scroll="handleScroll"
    >
      <slot v-bind="{ nextSlide, prevSlide }" />
    </div>
    <slot name="overlay" v-bind="{ nextSlide, prevSlide }" />
  </div>
</template>
