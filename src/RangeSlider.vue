<template>
    <div class="range-slider"></div>
</template>

<script>
import noUiSlider from 'nouislider';
import 'nouislider/dist/nouislider.css';

export default {
    name: 'RangeSlider',

    props: {
        behaviour: {
            type: String,
            default: 'drag',
        },
        connect: {
            type: Boolean,
            default: false,
        },
        direction: {
            type: String,
            default: 'ltr',
            validator: v => ['ltr', 'rtl'].includes(v),
        },
        manual: {
            type: Boolean,
            default: false,
        },
        margin: {
            type: Number,
            default: 1,
        },
        orientation: {
            type: String,
            default: 'horizontal',
            validator: v => ['horizontal', 'vertical'].includes(v),
        },
        step: {
            type: Number,
            default: 1,
            validator: Number.isInteger,
        },
        tooltips: {
            type: Boolean,
            default: false,
        },
        value: {
            type: Object,
            required: true,
            validator: v => Object.keys(v).length === 2
                && Object.keys(v).includes('min')
                && Object.keys(v).includes('max')
                && Number.isInteger(v.min) && Number.isInteger(v.max)
                && v.max > v.min,
        },
    },

    data: () => ({
        slider: null,
    }),

    computed: {
        options() {
            const {
                value, step, margin, connect, direction,
                orientation, behaviour, tooltips,
            } = this;

            return {
                range: value, step, margin, connect, direction, orientation,
                behaviour, tooltips, start: [value.min, value.max],
            };
        }
    },

    watch: {
        options: {
            handler(options) {
                this.slider.updateOptions(options);
            },
            deep: true,
        },
    },

    mounted() {
        this.init();
    },

    beforeDestroy() {
        this.slider.off('change');
        this.slider.off('set');
        this.slider.destroy();
    },

    methods: {
        init() {
            this.slider = noUiSlider.create(this.$el, this.options);
            this.slider.on('change', this.change);
            this.slider.on('slide', this.slide);
        },
        change([min, max]) {
            const int = Number.parseInt;

            if (!this.manual) {
                this.value.min = int(min);
                this.value.max = int(max);
            }

            this.$emit('change', { min: int(min), max: int(max) });
        },
        slide([min, max]) {
            const int = Number.parseInt;
            this.$emit('slide', {min: int(min), max: int(max)});
        },
    },
};
</script>

<style lang="scss">
    .noUi-target.noUi-horizontal {
        height: 7px;

        .noUi-handle {
            height: 18px;
            top: -7px;
            width: 12px;
            right: -5px;
            cursor: pointer;

            &:before {
                content: "";
                display: block;
                position: absolute;
                height: 12px;
                width: 1px;
                background: #E8E7E6;
                left: 3px;
                top: 2px;
            }

            &:after {
                left: 6px;
                height: 12px;
                top: 2px;
            }

            .noUi-tooltip {
                font-size: 10px;
                padding: 2px
            }
        }
    }
</style>