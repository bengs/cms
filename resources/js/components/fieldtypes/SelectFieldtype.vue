<template>
    <div class="flex">
        <v-select
            ref="input"
            class="flex-1"
            append-to-body
            :calculate-position="positionOptions"
            :name="name"
            :clearable="config.clearable"
            :disabled="config.disabled || isReadOnly || (config.multiple && limitReached)"
            :options="options"
            :placeholder="config.placeholder"
            :searchable="config.searchable || config.taggable"
            :taggable="config.taggable"
            :push-tags="config.push_tags"
            :multiple="config.multiple"
            :reset-on-options-change="resetOnOptionsChange"
            :close-on-select="true"
            :value="selectedOptions"
            :create-option="(value) => ({ value, label: value })"
            @input="vueSelectUpdated"
            @search:focus="$emit('focus')"
            @search:blur="$emit('blur')">
                <template #selected-option-container v-if="config.multiple"><i class="hidden"></i></template>
                <template #search="{ events, attributes }" v-if="config.multiple">
                    <input
                        :placeholder="config.placeholder"
                        class="vs__search"
                        type="search"
                        v-on="events"
                        v-bind="attributes"
                    >
                </template>
                <template #option="{ label }">
                    <div v-if="config.label_html" v-html="label"></div>
                    <template v-else v-text="label"></template>
                </template>
                <template #selected-option="{ label }">
                    <div v-if="config.label_html" v-html="label"></div>
                    <template v-else v-text="label"></template>
                </template>
                <template #no-options>
                    <div class="text-sm text-gray-700 text-left py-2 px-4" v-text="__('No options to choose from.')" />
                </template>
                <template #footer="{ deselect }" v-if="config.multiple">
                    <sortable-list
                        item-class="sortable-item"
                        handle-class="sortable-item"
                        :value="value"
                        :distance="5"
                        :mirror="false"
                        @input="update"
                    >
                    <div class="vs__selected-options-outside flex flex-wrap">
                        <span v-for="option in selectedOptions" :key="option.value" class="vs__selected mt-2 sortable-item">
                            <div v-if="config.label_html" v-html="option.label"></div>
                            <template v-else>{{ option.label }}</template>
                            <button v-if="!readOnly" @click="deselect(option)" type="button" :aria-label="__('Deselect option')" class="vs__deselect">
                                <span>×</span>
                            </button>
                            <button v-else type="button" class="vs__deselect">
                                <span class="text-gray-500">×</span>
                            </button>
                        </span>
                    </div>
                    </sortable-list>
                </template>
        </v-select>
        <div class="text-xs ml-2 mt-3" :class="limitIndicatorColor" v-if="config.max_items">
            <span v-text="currentLength"></span>/<span v-text="config.max_items"></span>
        </div>
    </div>
</template>

<style scoped>
    .draggable-source--is-dragging {
        @apply opacity-75 bg-transparent border-dashed
    }
</style>

<script>
import HasInputOptions from './HasInputOptions.js'
import { computePosition, offset, flip } from '@floating-ui/dom';
import { SortableList } from '../sortable/Sortable';


export default {

    mixins: [Fieldtype, HasInputOptions],

    components: {
        SortableList
    },

    computed: {
        selectedOptions() {
            let selections = this.value || [];
            if (typeof selections === 'string' || typeof selections === 'number') {
                selections = [selections];
            }
            return selections.map(value => {
                return _.findWhere(this.options, {value}) || { value, label: value };
            });
        },

        options() {
            return this.normalizeInputOptions(this.config.options);
        },

        replicatorPreview() {
            return this.selectedOptions.map(option => option.label).join(', ');
        },

        resetOnOptionsChange() {
            // Reset logic should only happen when the config value is true.
            // Nothing should be reset when it's false or undefined.
            if (this.config.reset_on_options_change !== true) return false;

            // Reset the value if the value doesn't exist in the new set of options.
            return (options, old, val) => {
                let opts = options.map(o => o.value);
                return !val.some(v => opts.includes(v.value));
            };
        },

        limitReached() {
            if (! this.config.max_items) return false;

            return this.currentLength >= this.config.max_items;
        },

        limitExceeded() {
            if (! this.config.max_items) return false;

            return this.currentLength > this.config.max_items;
        },

        currentLength() {
            if (this.value) {
                return (typeof this.value == 'string') ? 1 : this.value.length;
            }

            return 0;
        },

        limitIndicatorColor() {
            if (this.limitExceeded) {
                return 'text-red-500';
            } else if (this.limitReached) {
                return 'text-green-600';
            }

            return 'text-gray';
        }
    },

    methods: {
        focus() {
            this.$refs.input.focus();
        },

        vueSelectUpdated(value) {
            if (this.config.multiple) {
                this.update(value.map(v => v.value));
            } else {
                if (value) {
                    this.update(value.value)
                } else {
                    this.update(null);
                }
            }
        },

        positionOptions(dropdownList, component, { width }) {
            dropdownList.style.width = width

            computePosition(component.$refs.toggle, dropdownList, {
                placement: 'bottom',
                middleware: [
                    offset({ mainAxis: 0, crossAxis: -1 }),
                    flip(),
                ]
            }).then(({ x, y }) => {
                Object.assign(dropdownList.style, {
                    // Round to avoid blurry text
                    left: `${Math.round(x)}px`,
                    top: `${Math.round(y)}px`,
                });
            });
        },
    }
};
</script>
