<template>
  <div class="scroll-slide" :style="{ height: `${containerHeight}px` }">
    <div class="scroll-slide-sticky">
      <div
        ref="slot"
        class="scroll-slide-slot"
        :style="{ transform: `translate3d(${computedTransfrom}%, 0, 0)` }"
      >
        <slot v-bind="{ onReady }"></slot>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * @param {number} y - percentage
 * @param {number} x - whole
 * @return {number} - part
 */
const isOf = (p) => (x) => (p * x) / 100;

/**
 * @param {number} y - part
 * @param {number} x - whole
 * @return {number} - percentage
 */
const isWhatOf = (y) => (x) => (y / x) * 100;

let waitFrameTick = false;

export default {
    props: {
        startOffset: {
            type: Number,
            default: 33,
            validator: (value) => value >= 0 && value <= 100,
        },
        endOffset: {
            type: Number,
            default: -33,
            validator: (value) => value <= 0 && value >= -100,
        },
        speedMultiplier: {
            type: Number,
            default: 1.2, // bigger = slower
        },
    },

    data() {
        return {
            slotWidth: 0,
            clientHeight: 0,
            clientWidth: 0,
            pageTop: 0,
            slotTop: 0,
            slotBottom: 0,
            hasIntersectTop: false,
            hasIntersectBottom: false,
        };
    },

    computed: {
        containerHeight() {
            return this.slotWidth * this.speedMultiplier;
        },

        inViewport() {
            return this.hasIntersectTop && this.hasIntersectBottom;
        },

        hasScopedSlot() {
            return !this.$scopedSlots.default()[0].elm;
        },

        intersectedScroll() {
            if (!this.inViewport && this.hasIntersectTop)
                return this.containerHeight - this.clientHeight;
            if (!this.inViewport && this.hasIntersectBottom) return 0;

            return this.pageTop - this.slotTop;
        },

        computedStartOffset() {
            const offsetValue = isOf(this.startOffset)(this.clientWidth);
            const relativeOffset = isWhatOf(offsetValue)(this.slotWidth);
            return relativeOffset;
        },

        computedEndOffset() {
            const remainder = -100 - this.endOffset;
            const offsetValue = isOf(remainder)(this.clientWidth);
            const relativeOffset = isWhatOf(offsetValue)(this.slotWidth);
            return -100 - relativeOffset;
        },

        computedTransfrom() {
            const totalLength = Math.abs(
                this.computedEndOffset - this.computedStartOffset
            );
            const scrolled =
                (this.intersectedScroll * totalLength) /
                (this.containerHeight - this.clientHeight);
            return this.computedStartOffset - scrolled;
        },
    },

    mounted() {
        // Scoped slots should be used to expose the onReady in the template
        // to defer auto initialization.
        if (!this.hasScopedSlot) this.onReady();
        document.addEventListener('scroll', this.updateCoords, false);
    },

    beforeDestroy() {
        document.removeEventListener('scroll', this.updateCoords, false);
    },

    methods: {
        onReady() {
            const root = document.documentElement;
            this.slotWidth = this.$refs.slot.clientWidth;
            this.clientHeight = root.clientHeight;
            this.clientWidth = root.clientWidth;

            this.updateCoords();
        },

        updateCoords() {
            if (waitFrameTick) return;

            window.requestAnimationFrame(() => {
                const { top, bottom } = this.$el.getBoundingClientRect();
                this.pageTop = window.pageYOffset;
                this.slotTop = top + this.pageTop;
                this.slotBottom = bottom + this.pageTop;
                this.hasIntersectTop = this.slotTop <= this.pageTop;
                this.hasIntersectBottom =
                    this.slotBottom >= this.pageTop + this.clientHeight;

                waitFrameTick = false;
            });

            waitFrameTick = true;
        },
    },
};
</script>

<style lang="scss" scoped>
.scroll-slide {
    position: relative;
    width: 100%;
    margin: 500px 0;
}

.scroll-slide-sticky {
    position: sticky;
    top: 0;
    display: flex;
    align-items: center;
    width: 100%;
    height: 100vh;
    overflow: hidden;
}

.scroll-slide-slot {
    will-change: transform;
}
</style>