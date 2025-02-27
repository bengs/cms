<template>

    <div class="flex">
        <div class="page-move w-6" />
        <div class="flex items-center flex-1 p-2 ml-2 text-xs leading-normal">
            <div class="flex items-center flex-1" :class="{ 'opacity-50': isHidden || isInHiddenSection }">
                <template v-if="! isSection">
                    <i v-if="isAlreadySvg" class="w-4 h-4 mr-2" v-html="icon"></i>
                    <svg-icon v-else class="w-4 h-4 mr-2" :name="icon" />
                </template>

                <a
                    @click="$emit('edit', $event)"
                    :class="{ 'text-sm font-medium': isSection }"
                    v-text="__(item.text)" />

                <button
                    v-if="hasChildren && !isSection"
                    class="p-2 text-gray-600 hover:text-gray-700 transition duration-100 outline-none flex"
                    :class="{ '-rotate-90': !isOpen }"
                    @click="$emit('toggle-open')"
                >
                    <svg-icon name="micro/chevron-down-xs" class="h-1.5" />
                </button>

                <div v-if="item.collection" class="ml-4 flex items-center">
                    <svg-icon name="light/content-writing" class="w-4 h-4" />
                    <div class="ml-1">
                        <a :href="item.collection.create_url" v-text="__('Add')" />
                        <span class="text-gray">/</span>
                        <a :href="item.collection.edit_url" v-text="__('Edit')" />
                    </div>
                </div>
            </div>

            <div class="pr-2 flex items-center">
                <slot name="branch-icon" :branch="item" />

                <svg-icon v-if="isRenamedSection" class="inline-block w-4 h-4 text-gray-500" name="content-writing" v-tooltip="__('Renamed Section')" />
                <svg-icon v-else-if="isHidden" class="inline-block w-4 h-4 text-gray-500" name="hidden" v-tooltip="isSection ? __('Hidden Section') : __('Hidden Item')" />
                <svg-icon v-else-if="isPinnedAlias" class="inline-block w-4 h-4 text-gray-500" name="pin" v-tooltip="__('Pinned Item')" />
                <svg-icon v-else-if="isAlias" class="inline-block w-4 h-4 text-gray-500" name="duplicate-ids" v-tooltip="__('Alias Item')" />
                <svg-icon v-else-if="isMoved" class="inline-block w-4 text-gray-500" name="flip-vertical" v-tooltip="__('Moved Item')" />
                <svg-icon v-else-if="isModified" class="inline-block w-4 h-4 text-gray-500" name="content-writing" v-tooltip="__('Modified Item')" />
                <svg-icon v-else-if="isCustom" class="inline-block w-4 text-gray-500" name="user-edit" v-tooltip="isSection ? __('Custom Section') : __('Custom Item')" />

                <dropdown-list class="ml-4">
                    <slot name="branch-options"
                        :item="item"
                        :depth="depth"
                        :remove-branch="remove"
                    />
                </dropdown-list>
            </div>

        </div>
    </div>

</template>

<script>
import { data_get } from  '../../bootstrap/globals.js'

export default {

    props: {
        item: Object,
        parentSection: Object,
        depth: Number,
        root: Boolean,
        vm: Object,
        isOpen: Boolean,
        hasChildren: Boolean,
        disableSections: Boolean,
        topLevel: Boolean,
    },

    data() {
        return {
            editing: false,
        }
    },

    computed: {

        isSection() {
            if (this.disableSections) {
                return false;
            }

            return this.depth === 1;
        },

        title() {
            return this.item.title || this.item.entry_title || this.item.url;
        },

        icon() {
            return data_get(this.item.config, 'icon') || 'entries';
        },

        isAlreadySvg() {
            return this.icon.startsWith('<svg');
        },

        isRenamedSection() {
            return this.isSection && this.item.text !== data_get(this.item, 'config.display_original');
        },

        isHidden() {
            return data_get(this.item, 'manipulations.action') === '@hide';
        },

        isInHiddenSection() {
            return data_get(this.parentSection, 'manipulations.action') === '@hide';
        },

        isPinnedAlias() {
            return data_get(this.item, 'manipulations.action') === '@alias' && this.topLevel;
        },

        isAlias() {
            return data_get(this.item, 'manipulations.action') === '@alias';
        },

        isMoved() {
            return data_get(this.item, 'manipulations.action') === '@move';
        },

        isModified() {
            return data_get(this.item, 'manipulations.action') === '@modify';
        },

        isCustom() {
            return data_get(this.item, 'manipulations.action') === '@create';
        },

    },

    methods: {

        remove() {
            const store = this.item._vm.store;
            store.deleteNode(this.item);
            this.$emit('removed', store);
        },

    }

}
</script>
