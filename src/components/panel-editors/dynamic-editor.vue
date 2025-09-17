<template>
    <div class="block mt-6">
        <!-- Section selection header buttons -->
        <div class="flex gap-2 border-b pb-1 px-1.5" style="border-color: rgba(209, 213, 219, 1)">
            <!-- Text section button -->
            <button
                @click="() => changePanel('text')"
                :aria-label="$t('dynamic.textSection')"
                class="respected-standard-button respected-transparent-button respected-thin-button"
                :class="editingStatus === 'text' ? 'bg-gray-200' : ''"
                :style="editingStatus === 'text' ? 'font-weight: bold' : ''"
            >
                {{ $t('dynamic.textSection') }}
            </button>
            <!-- Panel collection button -->
            <button
                @click="() => changePanel('panels')"
                :aria-label="$t('dynamic.panel.collection') + ` (${panel.children.length})`"
                class="respected-standard-button respected-transparent-button respected-thin-button"
                :class="editingStatus !== 'text' ? 'bg-gray-200' : ''"
                :style="editingStatus !== 'text' ? 'font-weight: bold' : ''"
            >
                {{ $t('dynamic.panel.collection') + ` (${panel.children.length})` }}
            </button>
        </div>
        <!-- Text Section -->
        <div v-if="editingStatus === 'text'">
            <component :is="'text-editor'" key="text" :panel="panel" :lang="lang"></component>
        </div>
        <!-- Panel section -->
        <div v-if="editingStatus === 'panels'">
            <table-component
                :panel="panel"
                :panelItems="panel.children"
                :dynamicSelected="true"
                :editingSlide="editingSlide"
                v-model:newSlideName="newSlideName"
                v-model:newSlideType="newSlideType"
                @move-item-up="moveChildUp"
                @move-item-down="moveChildDown"
                @edit-item="switchSlide"
                @delete-item="removeSlide"
                @create-new-item="createNewSlide"
            />
            <!-- Panel editing area -->
            <div id="editor-area" v-if="editingSlide !== -1">
                <br />
                <hr />
                <br />
                <div class="flex justify-between items-center">
                    <!-- Header: Title, panel # and ID -->
                    <div class="flex flex-col">
                        <h2 class="font-bold text-xl">{{ $t('dynamic.panel.editor') }}</h2>
                        <p class="font-semibold text-md text-gray-500">
                            {{ `#${editingSlide + 1} - ${panel.children[editingSlide].id}` }}
                        </p>
                    </div>
                    <!-- Done button: closes panel editing area -->
                    <button
                        class="respected-standard-button respected-black-bg-button"
                        @click="switchSlide(-1)"
                        :aria-label="$t('editor.done')"
                    >
                        {{ $t('editor.done') }}
                    </button>
                </div>

                <!-- Actual panel editor -->
                <component
                    ref="slide"
                    :is="editors[determineEditorType(panel.children[editingSlide].panel)]"
                    :key="editingSlide + determineEditorType(panel.children[editingSlide].panel)"
                    :panel="panel.children[editingSlide].panel"
                    :lang="lang"
                    :centerSlide="centerSlide"
                    :dynamicSelected="dynamicSelected"
                    @slide-edit="$emit('slide-edit', 'Dynamic editor')"
                ></component>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Options, Prop, Vue } from 'vue-property-decorator';
import {
    BasePanel,
    BaseStartingConfig,
    DefaultConfigs,
    DynamicChildItem,
    DynamicPanel,
    PanelType
} from '@/definitions';
import { applyTextAlign } from '@/utils/styleUtils';

import ChartEditorV from './chart-editor.vue';
import ImageEditorV from './image-editor.vue';
import TextEditorV from './text-editor.vue';
import MapEditorV from './map-editor.vue';
import VideoEditorV from './video-editor.vue';
import SlideshowEditorV from './slideshow-editor.vue';
import TableComponentV from '../support/table-component.vue';
import { useProductStore } from '@/stores/productStore';

@Options({
    components: {
        'chart-editor': ChartEditorV,
        'image-editor': ImageEditorV,
        'text-editor': TextEditorV,
        'slideshow-editor': SlideshowEditorV,
        'dynamic-editor': DynamicEditorV,
        'map-editor': MapEditorV,
        'video-editor': VideoEditorV,
        'table-component': TableComponentV
    }
})
export default class DynamicEditorV extends Vue {
    @Prop() panel!: DynamicPanel;
    @Prop() lang!: string;
    @Prop() centerSlide!: boolean;
    @Prop() dynamicSelected!: boolean;

    productStore = useProductStore();

    editors: Record<string, string> = {
        text: 'text-editor',
        image: 'image-editor',
        slideshow: 'slideshow-editor',
        chart: 'chart-editor',
        map: 'map-editor',
        video: 'video-editor'
    };

    startingConfig: DefaultConfigs = JSON.parse(JSON.stringify(BaseStartingConfig));

    editingStatus = 'text';
    editingSlide = -1;

    newSlideName = '';
    newSlideType = 'text';

    moveChildUp(index: number): void {
        if (index === 0) {
            return;
        }
        const item = JSON.parse(JSON.stringify(this.panel.children[index]));
        this.panel.children.splice(index, 1);
        this.panel.children.splice(index - 1, 0, item);

        // Just in case we decide to allow it: handling edge case where a child
        // being edited is moved. In that case, ensure focus remains on that child
        if (this.editingSlide !== -1) {
            // If child being edited is current child
            if (this.editingSlide === index) {
                this.editingSlide -= 1;
                // If child being edited is previous child (shift it down)
            } else if (this.editingSlide === index - 1) {
                this.editingSlide += 1;
            }
        }
    }

    moveChildDown(index: number): void {
        if (index === this.panel.children.length - 1) {
            return;
        }

        const item = JSON.parse(JSON.stringify(this.panel.children[index]));
        this.panel.children.splice(index, 1);
        this.panel.children.splice(index + 1, 0, item);

        // Just in case we decide to allow it: handling edge case where a child
        // being edited is moved. In that case, ensure focus remains on that child
        if (this.editingSlide !== -1) {
            // If child being edited is current child
            if (this.editingSlide === index) {
                this.editingSlide += 1;
                // If child being edited is next child (shift it up)
            } else if (this.editingSlide === index + 1) {
                this.editingSlide -= 1;
            }
        }
    }

    changePanel(target: string): void {
        if (this.editingStatus !== 'text') {
            this.saveChanges();
        }
        this.editingStatus = target;
    }

    switchSlide(idx: number): void {
        if (idx === -1 && this.editingSlide !== -1) {
            // user pressed Done, so apply styles to the last active slide
            const editedPanel = this.panel.children[this.editingSlide].panel;
            applyTextAlign(editedPanel, this.centerSlide, this.dynamicSelected);
            this.editingSlide = -1;
        } else {
            // Save slide changes if necessary and switch to the newly selected slide.
            this.saveChanges();
            this.editingSlide = idx;

            // After switching the edit status, scroll to the add button.
            this.$nextTick(() => {
                document.getElementById('editor-area')?.scrollIntoView({ block: 'nearest', behavior: 'smooth' });
            });
        }
    }

    removeSlide(panel: BasePanel, index?: number): void {
        // Update source counts based on which panel is removed.
        this.productStore.removeSourceCounts(panel);

        if (index !== undefined) {
            // Remove the panel itself.
            this.panel.children = this.panel.children.filter((panel: DynamicChildItem, idx: number) => idx !== index);

            // If the slide being removed is the currently selected slide, unselect it.
            if (this.editingSlide === index) {
                this.editingSlide = -1;
            }
        }
    }

    createNewSlide(): void {
        if (!this.newSlideName) return;

        const newConfig = {
            id: this.newSlideName,
            panel: JSON.parse(JSON.stringify(this.startingConfig[this.newSlideType as keyof DefaultConfigs]))
        };
        this.editingSlide = this.panel.children.length;
        // this.editingMode = false;
        this.newSlideName = '';
        this.panel.children.push(newConfig);
    }

    determineEditorType(panel: BasePanel): string {
        // Determine whether the slideshow consists of only charts. If so, display the chart editor.
        if (panel.type === 'slideshowChart') return PanelType.Chart;

        // Determine whether the slideshow consists of only images. If so, display the image editor.
        if (panel.type === 'slideshowImage') return PanelType.Image;

        if (panel.type !== PanelType.Slideshow) return panel.type;

        // Otherwise display the slideshow editor.
        return PanelType.Slideshow;
    }

    saveChanges(): void {
        if (
            this.editingSlide !== -1 &&
            this.$refs.slide !== undefined &&
            typeof (this.$refs.slide as ImageEditorV | ChartEditorV)?.saveChanges === 'function'
        ) {
            (this.$refs.slide as ImageEditorV | ChartEditorV).saveChanges();
        }
    }
}
</script>

<style lang="scss"></style>
