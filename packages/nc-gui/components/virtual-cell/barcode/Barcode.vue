<script setup lang="ts">
import type { ComputedRef } from 'vue'
import JsBarcodeWrapper from './JsBarcodeWrapper.vue'
import { RowHeightInj, computed, inject, ref } from '#imports'

const maxNumberOfAllowedCharsForBarcodeValue = 100

const cellValue = inject(CellValueInj)

const column = inject(ColumnInj)

const barcodeValue: ComputedRef<string> = computed(() => String(cellValue?.value || ''))

const tooManyCharsForBarcode = computed(() => barcodeValue.value.length > maxNumberOfAllowedCharsForBarcodeValue)

const modalVisible = ref(false)

const isExpandedFormOpen = inject(IsExpandedFormOpenInj, ref(false))

const showBarcodeModal = () => {
  modalVisible.value = true
}

const barcodeMeta = computed(() => {
  return {
    barcodeFormat: 'CODE128',
    ...parseProp(column?.value?.meta),
  }
})

const handleModalOkClick = () => (modalVisible.value = false)

const showBarcode = computed(() => barcodeValue?.value.length > 0 && !tooManyCharsForBarcode.value)

const { showEditNonEditableFieldWarning, showClearNonEditableFieldWarning } = useShowNotEditableWarning()

const rowHeight = inject(RowHeightInj, ref(undefined))
</script>

<template>
  <a-modal
    v-model:visible="modalVisible"
    :class="{ active: modalVisible }"
    wrap-class-name="nc-barcode-large"
    :body-style="{ padding: '0px' }"
    :footer="null"
    @ok="handleModalOkClick"
  >
    <JsBarcodeWrapper
      v-if="showBarcode"
      :barcode-value="barcodeValue"
      :barcode-format="barcodeMeta.barcodeFormat"
      show-download
    />
  </a-modal>
  <div
    v-if="!tooManyCharsForBarcode"
    class="flex w-full items-center barcode-wrapper"
    :class="{
      'justify-start ml-2': isExpandedFormOpen,
      'justify-center': !isExpandedFormOpen,
    }"
  >
    <JsBarcodeWrapper
      v-if="showBarcode && rowHeight"
      :barcode-value="barcodeValue"
      tabindex="-1"
      :barcode-format="barcodeMeta.barcodeFormat"
      :custom-style="{ height: rowHeight ? `${rowHeight * 1.8}rem` : `1.8rem` }"
      @on-click-barcode="showBarcodeModal"
    >
      <template #barcodeRenderError>
        <div class="text-left text-wrap mt-2 text-[#e65100] text-xs" data-testid="barcode-invalid-input-message">
          {{ $t('msg.warning.barcode.renderError') }}
        </div>
      </template>
    </JsBarcodeWrapper>
    <JsBarcodeWrapper
      v-else-if="showBarcode"
      tabindex="-1"
      :barcode-value="barcodeValue"
      :barcode-format="barcodeMeta.barcodeFormat"
      @on-click-barcode="showBarcodeModal"
    >
      <template #barcodeRenderError>
        <div class="text-left text-wrap mt-2 text-[#e65100] text-xs" data-testid="barcode-invalid-input-message">
          {{ $t('msg.warning.barcode.renderError') }}
        </div>
      </template>
    </JsBarcodeWrapper>
  </div>

  <div v-if="tooManyCharsForBarcode" class="text-left text-wrap mt-2 text-[#e65100] text-xs">
    {{ $t('labels.barcodeValueTooLong') }}
  </div>
  <div v-if="showEditNonEditableFieldWarning" class="text-left text-wrap mt-2 text-[#e65100] text-xs">
    {{ $t('msg.warning.nonEditableFields.computedFieldUnableToClear') }}
  </div>
  <div v-if="showClearNonEditableFieldWarning" class="text-left text-wrap mt-2 text-[#e65100] text-xs">
    {{ $t('msg.warning.nonEditableFields.barcodeFieldsCannotBeDirectlyChanged') }}
  </div>
</template>

<style lang="scss" scoped>
.barcode-wrapper {
  & > div {
    @apply max-w-8.2rem;
  }
}
</style>
