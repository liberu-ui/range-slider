<template>
    <div class="range-slider"/>
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
        modelValue: {
            type: Object,
            required: true,
            validator: v => Object.keys(v).length === 2
                && Object.keys(v).includes('min')
                && Object.keys(v).includes('max')
                && Number.isInteger(v.min) && Number.isInteger(v.max)
                && v.max > v.min,
        },
    },

    emits: ['change', 'slide'],

    data: () => ({
        slider: null,
    }),

    computed: {
        options() {
            return {
                range: this.modelValue,
                step: this.step,
                margin: this.margin,
                connect: this.connect,
                direction: this.direction,
                orientation: this.orientation,
                behaviour: this.behaviour,
                tooltips: this.tooltips,
                start: [this.modelValue.min, this.modelValue.max],
            };
        },
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

    beforeUnmount() {
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

            if (this.modelValue.min === int(min) && this.modelValue.max === int(max)) {
                return;
            }

            if (!this.manual) {
                this.modelValue.min = int(min);
                this.modelValue.max = int(max);
            }

            this.$emit('change', { min: int(min), max: int(max) });
        },
        slide([min, max]) {
            const int = Number.parseInt;
            this.$emit('slide', { min: int(min), max: int(max) });
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
                padding: 2px;
            }
        }
    }
</style>
