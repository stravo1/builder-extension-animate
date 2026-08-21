<template>
	<div class="thin-slider">
		<Slider v-model="value" :min="min" :max="max" :step="step">
			<template #label>
				<span class="flex items-baseline justify-between text-p-sm text-ink-gray-6">
					{{ label }}
					<span class="text-p-xs tabular-nums text-ink-gray-5">{{ modelValue }} ms</span>
				</span>
			</template>
		</Slider>
	</div>
</template>

<script setup>
import { Slider } from "frappe-ui";
import { computed } from "vue";

const props = defineProps({
	label: { type: String, required: true },
	modelValue: { type: Number, required: true },
	min: { type: Number, required: true },
	max: { type: Number, required: true },
	step: { type: Number, default: 1 },
});
const emit = defineEmits(["update:modelValue"]);

const value = computed({
	get: () => [props.modelValue],
	set: ([next]) => emit("update:modelValue", next),
});
</script>

<style scoped>
.thin-slider :deep([data-slot="control"] > :first-child) {
	height: 2px;
}

.thin-slider :deep([role="slider"]) {
	height: 10px;
	width: 10px;
}
</style>
