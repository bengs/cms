<template>

    <div>

        <tabs
            :initial-tabs="tabs"
            :require-section="config.require_set"
            :can-define-localizable="false"
            :add-tab-text="__('Add Set Group')"
            :edit-tab-text="__('Edit Set Group')"
            :new-tab-text="__('New Set Group')"
            :add-section-text="__('Add Set')"
            :edit-section-text="__('Edit Set')"
            :new-section-text="__('New Set')"
            show-tab-instructions-field
            show-section-handle-field
            @updated="tabsUpdated"
        />

    </div>

</template>

<script>
import Tabs from '../../blueprints/Tabs.vue';

export default {

    mixins: [Fieldtype],

    components: {
        Tabs
    },

    data() {
        return {
            tabs: this.value
        }
    },

    methods: {

        tabsUpdated(tabs) {
            this.update(tabs);
        }

    }

}
</script>
