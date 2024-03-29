<template>
  <div
    class="breaky"
    :class="`color-scheme-${colorScheme}`"
  >
    <div
      v-show="!TOGGLE_ME_TO_HIDE_BREAKY"
      ref="breaky"
      class="card text-xs fixed flex p-2 z-50 shadow cursor-pointer antialiased font-bold leading-normal tracking-wide"
      :class="[
        draggableTransitionClasses,
        {
          'flex-col-reverse': currentPosition.includes('top'),
          'flex-col': currentPosition.includes('bottom'),
        },
      ]"
      @click.stop="!noExpand ? (expanded = !expanded) : null"
    >
      <TransitionExpand>
        <div
          v-show="expanded"
          class="pt-1 pb-2 relative"
        >
          <!-- Selected Panel -->
          <span
            v-show="foundBreakpoint !== 0"
            class="absolute h-selected w-full bg-selected rounded-lg ease-out transition-transform duration-200"
            :style="{ transform: `translateY(calc(100% * ${selected}))` }"
          />
          <!-- END Selected Panel -->
          <ul class="relative">
            <li
              v-for="(bp, name, index) in mappedBreakpoints"
              :key="index"
              class="flex justify-between py-2 px-4"
              :class="{ 'opacity-50': selected !== index }"
            >
              <span>{{ name }} </span>
              <span class="ml-5">{{ bp }}px</span>
            </li>
          </ul>
        </div>
      </TransitionExpand>

      <div
        class="current-breakpoint transition duration-300 text-center border-2 border-transparent py-2 px-4 rounded-full flex items-center justify-around"
        :class="{ 'border-opacity-30': !expanded }"
      >
        <CurrentScreenIcon :screen-width="screenWidth" />
        {{ currentBreakpoint }} - {{ screenWidth }}px
      </div>
    </div>
  </div>
</template>

<script>
import throttle from 'lodash/throttle'
import interact from 'interactjs'
import TransitionExpand from './TransitionExpand'
import CurrentScreenIcon from './CurrentScreenIcon'

export default {
  components: {
    TransitionExpand,
    CurrentScreenIcon
  },

  props: {
    breakpoints: {
      type: Object,
      required: true
    },
    startingPosition: {
      type: String,
      default: 'bottomRight'
    },
    colorScheme: {
      type: String,
      default: 'auto'
    }
  },

  data () {
    return {
      TOGGLE_ME_TO_HIDE_BREAKY: false,
      expanded: false,
      noExpand: false,
      screenWidth: window.innerWidth,
      screenHeight: window.innerHeight,
      currentPosition: this.startingPosition,
      draggableTransitionClasses: ['transition-all', 'duration-300']
    }
  },

  computed: {
    /**
     * Convert the breakpoints to integers and filter non-pixel values
     * example: 1024px => 1024
     */
    mappedBreakpoints () {
      const mappedScreens = {}

      Object.keys(this.breakpoints).forEach(
        (key) => {
          if (typeof this.breakpoints[key] === 'string') {
            const match = this.breakpoints[key].match(/(\d+)px/)
            if (match) {
              mappedScreens[key] = parseInt(match[1])
            }
          }
        }
      )

      return mappedScreens
    },

    /**
     * Sort mapped breakpoints based on its values
     */
    sortedBreakpoints () {
      return Object.keys(this.mappedBreakpoints).sort((a, b) => {
        if (this.mappedBreakpoints[a] < this.mappedBreakpoints[b]) {
          return -1
        }
        if (this.mappedBreakpoints[a] > this.mappedBreakpoints[b]) {
          return 1
        }
        return 0
      })
    },

    /**
     * Get the index of the current breakpoint based on the screen width
     */
    foundBreakpoint () {
      return this.sortedBreakpoints.findIndex(
        (key) => this.mappedBreakpoints[key] > this.screenWidth
      )
    },

    /**
     * Get the index of the current active breakpoint
     */
    selected () {
      return this.sortedBreakpoints.findIndex(
        (bp) => bp === this.currentBreakpoint
      )
    },

    /**
     * Evaluate the current breakpoint based on the
     * browser screen width
     */
    currentBreakpoint () {
      // check if the screen is smaller than the smallest
      // defined screen in the tailwind config
      if (this.foundBreakpoint === 0) {
        return `< ${this.mappedBreakpoints[this.sortedBreakpoints[0]]}px`
      }

      // when no breakpoint has been found take the highest
      if (this.foundBreakpoint === -1) {
        return this.sortedBreakpoints[this.sortedBreakpoints.length - 1]
      }

      // set the found breakpoint
      return this.sortedBreakpoints[this.foundBreakpoint - 1]
    },

    /**
     * Get the elements positioning offset
     */
    offset () {
      return {
        x: 32,
        y: 24
      }
    },

    /**
     * Get snap points based on screen size and offset
     */
    topLeft () {
      return { name: 'topLeft', x: this.offset.x, y: this.offset.y }
    },
    topRight () {
      return {
        name: 'topRight',
        x: this.screenWidth - this.offset.x,
        y: this.offset.y
      }
    },
    bottomLeft () {
      return {
        name: 'bottomLeft',
        x: this.offset.x,
        y: this.screenHeight - this.offset.y
      }
    },
    bottomRight () {
      return {
        name: 'bottomRight',
        x: this.screenWidth - this.offset.x,
        y: this.screenHeight - this.offset.y
      }
    },
    snapPoints () {
      return [this.topLeft, this.topRight, this.bottomLeft, this.bottomRight]
    }
  },

  mounted () {
    this.resizeHandler()

    window.addEventListener('resize', this.resizeHandler)

    this.$on('hook:beforeDestroy', () => {
      window.removeEventListener('resize', this.resizeHandler)
    })

    this.applyStartingPosition()
    this.initInteract()
  },

  methods: {
    /**
     *  Apply the starting position passed through as a prop
     */
    applyStartingPosition () {
      if (typeof this[this.startingPosition] === 'object') {
        // get the elements size
        const w = this.$refs.breaky.clientWidth
        const h = this.$refs.breaky.clientHeight
        // get target coordinates
        const { x, y } = this[this.startingPosition]

        this.updatePosition(this.$refs.breaky, x, y, w, h)
      }
    },

    /**
     *  Update the reactive property of screen width and height
     */
    resizeHandler: throttle(function () {
      this.screenWidth = window.innerWidth
      this.screenHeight = window.innerHeight
    }, 100),

    /**
     *  Update the breaky elements position
     */
    updatePosition (target, x, y, w, h) {
      if (x > this.screenWidth / 2) {
        target.style.left = 'auto'
        target.style.right = this.screenWidth - x - w / 2 + 'px'
      } else {
        target.style.left = x - w / 2 + 'px'
        target.style.right = 'auto'
      }

      if (y > this.screenHeight / 2) {
        target.style.top = 'auto'
        target.style.bottom = this.screenHeight - y - h / 2 + 'px'
      } else {
        target.style.top = y - h / 2 + 'px'
        target.style.bottom = 'auto'
      }
    },

    /**
     *  Get the closest snappoint to a coordinate
     */
    getClosestSnapPoint (x, y) {
      // calculate distance to each snappoint
      const distances = this.snapPoints.map((point) =>
        Math.sqrt((x - point.x) ** 2 + (y - point.y) ** 2)
      )

      // get the shortest distance
      const closest = [...distances].sort((a, b) =>
        a > b ? 1 : a < b ? -1 : 0
      )[0]
      // get index of the shortest distance in order to get its coordinates
      const closestIndex = distances.indexOf(closest)

      // get the closest snappoints coordinates
      const closestSnapPoint = this.snapPoints[closestIndex]
      const closestX = closestSnapPoint.x
      const closestY = closestSnapPoint.y
      const closestName = closestSnapPoint.name

      return { x: closestX, y: closestY, name: closestName }
    },

    /**
     *  Initialize the breaky element to be draggable
     */
    initInteract () {
      // get size of breaky element
      const w = this.$refs.breaky.clientWidth
      const h = this.$refs.breaky.clientHeight

      interact(this.$refs.breaky).draggable({
        onstart: (event) => {
          // prevent breaky from expanding and transitioning while dragging
          this.noExpand = true
          event.target.classList.remove(...this.draggableTransitionClasses)
        },

        onend: (event) => {
          // allow breaky to expand and transition again
          setTimeout(() => (this.noExpand = false), 0)
          event.target.classList.add(...this.draggableTransitionClasses)

          // get the closest snappoint
          const { x, y, name } = this.getClosestSnapPoint(
            event.pageX,
            event.pageY
          )
          this.currentPosition = name

          // update the breaky elements position
          this.updatePosition(event.target, x, y, w, h)
        },

        onmove: (event) => {
          // update the elements position based on its current size.
          // the size may have changed if the element has been extended before this method is called.
          // this matters if we want the element to be dragged from the center
          this.updatePosition(
            event.target,
            event.pageX,
            event.pageY,
            event.target.clientWidth,
            event.target.clientHeight
          )
        }
      })
    }
  }
}
</script>

<style scoped lang="scss">
@import '../assets/scss/mixins/color-scheme.scss';

.color-scheme-auto {
  /* Light mode */
  @media (prefers-color-scheme: light) {
    @include light-card;
  }

  /* Dark mode */
  @media (prefers-color-scheme: dark) {
    @include dark-card;
  }
}

.color-scheme-light {
  @include light-card;
}

.color-scheme-dark {
  @include dark-card;
}

.breaky {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
}

@media print {
  .breaky {
    @apply hidden;
  }
}

.card {
  min-width: 170px;
  border-radius: 1.75rem;
  animation: fadeIn 0.25s forwards;

  touch-action: none;
  user-select: none;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.h-selected {
  height: 34px;
}
</style>
