<template>
    <div class="w-full lg:w-4/5 max-h-96 overflow-y-auto mt-5 border border-gray-400 rounded-md">
        <!-- Panel collection table -->
        <table class="w-full border-separate">
            <thead class="bg-white sticky top-0 z-10">
                <tr style="top: 2px" class="table-header sticky z-20">
                    <!-- Header: Move panel buttons -->
                    <th
                        style="width: 10%; text-align: left !important"
                        class="rounded-tl"
                        :aria-label="$t('editor.slideshow.label.slideNumber')"
                    ></th>
                    <!-- Header: Panel # and ID for dynmaic panels, and Slideshow # for slideshow panels -->
                    <th
                        style="width: 40%; text-align: left !important"
                        :aria-label="
                            dynamicSelected ? $t('dynamic.panel.id') : $t('editor.slideshow.label.slideNumber')
                        "
                    >
                        <span class="mx-2">{{
                            dynamicSelected ? $t('dynamic.panel.id') : $t('editor.slideshow.label.slideNumber')
                        }}</span>
                    </th>
                    <!-- Header: Panel tyoe  -->
                    <th style="width: 20%">
                        <span class="mx-2">{{
                            dynamicSelected ? $t('dynamic.panel.type') : $t('editor.slideshow.label.type')
                        }}</span>
                    </th>

                    <!-- Header: Panel actions -->
                    <th style="width: 30%" class="rounded-tr">
                        <span class="mx-2">{{
                            dynamicSelected ? $t('dynamic.panel.actions') : $t('dynamic.panel.actions')
                        }}</span>
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr
                    class="table-contents"
                    style="cursor: default !important"
                    v-for="(item, idx) in panelItems"
                    :key="idx"
                    :class="{
                        'bg-gray-100': idx % 2 !== 0,
                        'bg-blue-200 hover-editing':
                            editingStatus !== 'none' && editingStatus !== 'create' && editingSlide === idx
                    }"
                >
                    <!-- Move panel buttons -->
                    <td
                        class="px-0.5 flex flex-col lg:flex-row gap-0.5 justify-center items-center"
                        :class="{ 'rounded-bl': idx === panelItems.length - 1 }"
                    >
                        <!-- Move panel up button -->
                        <button
                            :disabled="idx === 0 || editingSlide !== -1"
                            @click="$emit('move-item-up', idx)"
                            :aria-label="$t('editor.slides.toc.moveSlideUp')"
                            style="border: none !important; padding-left: 0.375rem; padding-right: 0.375rem"
                            class="respected-standard-button respected-transparent-button"
                            v-tippy="{
                                delay: '200',
                                placement: 'top',
                                content: $t('editor.slides.toc.moveSlideUp'),
                                touch: ['hold', 500]
                            }"
                        >
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                xmlns:xlink="http://www.w3.org/1999/xlink"
                                x="0px"
                                y="0px"
                                viewBox="0 0 122.88 66.91"
                                style="enable-background: new 0 0 122.88 66.91"
                                xml:space="preserve"
                                height="14"
                                width="14"
                                class="fill-current"
                            >
                                <g>
                                    <path
                                        d="M11.68,64.96c-2.72,2.65-7.08,2.59-9.73-0.14c-2.65-2.72-2.59-7.08,0.13-9.73L56.87,1.97l4.8,4.93l-4.81-4.95 c2.74-2.65,7.1-2.58,9.76,0.15c0.08,0.08,0.15,0.16,0.23,0.24L120.8,55.1c2.72,2.65,2.78,7.01,0.13,9.73 c-2.65,2.72-7,2.78-9.73,0.14L61.65,16.5L11.68,64.96L11.68,64.96z"
                                    />
                                </g>
                            </svg>
                        </button>
                        <!-- Move panel down button -->
                        <button
                            :disabled="idx === panelItems.length - 1 || editingSlide !== -1"
                            @click="$emit('move-item-down', idx)"
                            :aria-label="$t('editor.slides.toc.moveSlideDown')"
                            style="border: none !important; padding-left: 0.375rem; padding-right: 0.375rem"
                            class="respected-standard-button respected-transparent-button rotate-180 transform"
                            v-tippy="{
                                delay: '200',
                                placement: 'top',
                                content: $t('editor.slides.toc.moveSlideDown'),
                                touch: ['hold', 500]
                            }"
                        >
                            <svg
                                xmlns="http://www.w3.org/2000/svg"
                                xmlns:xlink="http://www.w3.org/1999/xlink"
                                x="0px"
                                y="0px"
                                viewBox="0 0 122.88 66.91"
                                style="enable-background: new 0 0 122.88 66.91"
                                xml:space="preserve"
                                height="14"
                                width="14"
                                class="fill-current"
                            >
                                <g>
                                    <path
                                        d="M11.68,64.96c-2.72,2.65-7.08,2.59-9.73-0.14c-2.65-2.72-2.59-7.08,0.13-9.73L56.87,1.97l4.8,4.93l-4.81-4.95 c2.74-2.65,7.1-2.58,9.76,0.15c0.08,0.08,0.15,0.16,0.23,0.24L120.8,55.1c2.72,2.65,2.78,7.01,0.13,9.73 c-2.65,2.72-7,2.78-9.73,0.14L61.65,16.5L11.68,64.96L11.68,64.96z"
                                    />
                                </g>
                            </svg>
                        </button>
                    </td>
                    <!-- Panel # and ID / Slideshow #  -->
                    <td style="text-align: left !important" class="truncate px-2">
                        <div class="px-2">
                            <span class="text-gray-600">{{ `${idx + 1}. ` }}</span>
                            {{ dynamicSelected ? item.id : (item as any).title || $t('editor.slideshow.noTitle') }}
                        </div>
                    </td>
                    <!-- Panel/slide type -->
                    <td>
                        <span class="mx-2"
                            >{{ $t(`editor.slide.panel.type.${dynamicSelected ? item.panel.type : item.type}`) }}
                        </span>
                    </td>

                    <!-- Panel actions -->
                    <td :class="{ 'rounded-br': idx === panelItems.length - 1 }">
                        <!-- Edit panel -->
                        <span
                            @click="$emit('edit-item', idx)"
                            @keydown.enter="$emit('edit-item', idx)"
                            class="slideshow-text-button underline cursor-pointer rounded-sm"
                            tabindex="0"
                            >{{ $t('editor.chart.label.edit') }}</span
                        >
                        |
                        <!-- Delete panel -->
                        <span
                            @click="
                                () =>
                                    editingSlide === -1 &&
                                    (panel.type !== 'slideshow'
                                        ? $vfm.open(`delete-item-${idx}`)
                                        : $emit('delete-item', item, idx))
                            "
                            @keydown.enter="
                                editingSlide === -1 &&
                                (panel.type !== 'slideshow'
                                    ? $vfm.open(`delete-item-${idx}`)
                                    : $emit('delete-item', item, idx))
                            "
                            class="slideshow-text-button underline rounded-sm"
                            :class="[
                                editingSlide === -1 ? 'text-red-700 cursor-pointer' : 'text-gray-600 cursor-not-allowed'
                            ]"
                            tabindex="0"
                            >{{ $t('editor.remove') }}</span
                        >
                    </td>
                    <!-- confirm deleting slide -->
                    <ConfirmationModalV
                        :name="`delete-item-${idx}`"
                        :message="$t('dynamic.panel.remove')"
                        @ok="() => $emit(`delete-item`, item as BasePanel, idx)"
                    />
                </tr>
                <!-- Add new panel for dynamic tables -->
                <tr v-if="dynamicSelected" class="table-add-row sticky bottom-1 z-20 bg-white">
                    <td></td>
                    <!-- New panel ID input -->
                    <td class="flex flex-col items-start">
                        <input
                            v-model="localNewSlideName"
                            id="panelId"
                            class="respected-standard-input"
                            type="text"
                            :placeholder="$t('dynamic.panel.enterID')"
                            :aria-label="$t('dynamic.panel.enterID')"
                        />
                        <p v-if="idUsed" class="text-red-500">{{ $t('dynamic.panel.idTaken') }}</p>
                    </td>
                    <!-- New panel type input -->
                    <td>
                        <select
                            v-model="localNewSlideType"
                            class="rounded shadow w-48"
                            style="
                                width: 80% !important;
                                text-align-last: center !important;
                                justify-self: center !important;
                                text-align: start !important;
                                padding-left: 1rem !important;
                            "
                        >
                            <option v-for="thing in Object.keys(editors)" :key="thing" :value="thing">
                                {{ thing }}
                            </option>
                        </select>
                    </td>
                    <!-- Confirm new panel creation button -->
                    <td>
                        <button
                            class="respected-standard-button respected-gray-border-button respected-thin-button justify-self-center"
                            @click="$emit('create-new-item')"
                            :aria-label="$t('dynamic.panel.add')"
                            :disabled="idUsed || !newSlideName"
                        >
                            {{ $t('dynamic.panel.add') }}
                        </button>
                    </td>
                </tr>
                <tr v-if="panelItems.length === 0 && !dynamicSelected">
                    <!-- Empty/no slide prompt for slideshows -->
                    <td class="self-center text-center italic" colspan="4">
                        {{ $t('editor.slideshow.noSlides') }}
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
    <!-- add item button for slideshows -->
    <div v-if="!dynamicSelected">
        <button
            class="respected-standard-button w-full lg:w-4/5 max-h-96 bg-gray-100 border border-gray-400 hover:bg-gray-200"
            style="margin-top: 1.25rem; justify-content: start !important"
            @click="$emit('change-edit-status')"
        >
            <svg
                height="18px"
                width="18px"
                viewBox="0 0 23 21"
                xmlns="http://www.w3.org/2000/svg"
                v-if="editingStatus !== 'create'"
            >
                <path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z" />
            </svg>
            <svg
                class="fill-current"
                height="18px"
                width="18px"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 352 512"
                v-else
            >
                <path
                    d="M242.72 256l100.07-100.07c12.28-12.28 12.28-32.19 0-44.48l-22.24-22.24c-12.28-12.28-32.19-12.28-44.48 0L176 189.28 75.93 89.21c-12.28-12.28-32.19-12.28-44.48 0L9.21 111.45c-12.28 12.28-12.28 32.19 0 44.48L109.28 256 9.21 356.07c-12.28 12.28-12.28 32.19 0 44.48l22.24 22.24c12.28 12.28 32.2 12.28 44.48 0L176 322.72l100.07 100.07c12.28 12.28 32.2 12.28 44.48 0l22.24-22.24c12.28-12.28 12.28-32.19 0-44.48L242.72 256z"
                ></path>
            </svg>
            <span class="px-2">
                {{ editingStatus === 'create' ? $t('editor.cancel') : $t('editor.slideshow.label.create') }}
            </span>
        </button>
    </div>
</template>

<script setup lang="ts">
import { BasePanel, DynamicChildItem, DynamicPanel, SlideshowPanel } from '@/definitions';

import ConfirmationModalV from '../support/confirmation-modal.vue';
import { computed } from 'vue';

// =========================================
// Component props and emits

const props = defineProps<{
    panel: SlideshowPanel | DynamicPanel;
    panelItems: Array<any>;
    dynamicSelected: boolean;
    editingSlide: number;
    editingStatus?: string;
    newSlideName?: string;
    newSlideType?: string;
}>();

const emit = defineEmits([
    'move-item-up',
    'move-item-down',
    'edit-item',
    'delete-item',
    'create-new-item',
    'change-edit-status',
    'update:newSlideName',
    'update:newSlideType'
]);

// =========================================
// Definitions

const editors: Record<string, string> = {
    text: 'text-editor',
    image: 'image-editor',
    slideshow: 'slideshow-editor',
    chart: 'chart-editor',
    map: 'map-editor',
    video: 'video-editor'
};

const idUsed = computed(() => props.panelItems.some((ch: DynamicChildItem) => ch.id === props.newSlideName));

const localNewSlideName = computed({
    get: () => props.newSlideName,
    set: (val: string) => emit('update:newSlideName', val)
});

const localNewSlideType = computed({
    get: () => props.newSlideType,
    set: (val: string) => emit('update:newSlideType', val)
});
</script>

<style lang="scss" scoped>
label {
    text-align: left !important;
    width: fit-content !important;
}

.table-header th {
    text-align: center;
    background-color: #ddd;
    padding: 5px 10px;
}

.table-contents td {
    text-align: center;
    padding: 5px 10px;
}

.table-contents:hover {
    background-color: #eee;
    cursor: pointer;
}

.table-add-row td {
    vertical-align: top;
    text-align: center;
    border-top: 1px solid #ddd;
    padding: 5px;
    padding-bottom: 0 !important;
}

.table-add-row input[type='text'],
.table-add-row select,
.table-add-row button {
    width: 250px !important;
    max-width: 100% !important;
    padding-top: 3px !important;
    padding-bottom: 3px !important;
}

.hover-editing:hover {
    background-color: rgb(219, 234, 254);
}

select {
    border: 1px solid #a1a1a1;
    background: white;
    padding: 0.25rem 0.5rem;
}

.slideshow-text-button:focus {
    outline: 2px solid royalblue;
    z-index: 2;
    outline-offset: 2px;
    transition-duration: 0.075s;
}
</style>
