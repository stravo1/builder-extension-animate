<template>
	<section class="flex flex-col gap-2 rounded border border-outline-gray-2 bg-surface-gray-1 p-4">
		<svg viewBox="-12 -12 184 124" class="h-24 w-full rounded bg-surface-base" aria-label="Cubic bezier curve preview">
			<path d="M12 88 H148 M12 88 V12" class="stroke-outline-gray-2" fill="none" />
			<path d="M12 88 L148 12" class="stroke-outline-gray-3" fill="none" stroke-dasharray="3 3" />
			<path :d="curve" class="stroke-ink-gray-8" fill="none" stroke-width="2" />
			<path :d="handles" class="stroke-outline-gray-4" fill="none" stroke-dasharray="3 3" />
			<circle cx="12" cy="88" r="2" class="fill-surface-gray-5" />
			<circle cx="148" cy="12" r="2" class="fill-surface-gray-5" />
			<circle :cx="pointX(0)" :cy="pointY(1)" r="3" class="fill-surface-gray-7" />
			<circle :cx="pointX(2)" :cy="pointY(3)" r="3" class="fill-surface-gray-7" />
		</svg>

		<div class="grid grid-cols-2 gap-x-3 gap-y-1">
			<div v-for="control in controls" :key="control.index" class="thin-slider">
				<Slider
					:model-value="[points[control.index]]"
					:label="control.label"
					:min="control.min"
					:max="control.max"
					:step="0.01"
					@update:model-value="([value]) => update(control.index, value)" />
			</div>
		</div>
	</section>
</template>

<script setup>
import { Slider } from "frappe-ui";
import { computed } from "vue";
import { EASING_VALUES } from "../runtime.js";

const props = defineProps({ modelValue: { type: String, required: true } });
const emit = defineEmits(["update:modelValue"]);

const parseCurve = (value) => {
	const source = EASING_VALUES[value] || value;
	const numbers = source.match(/^cubic-bezier\(([^)]+)\)$/)?.[1].split(",").map(Number);
	return numbers?.length === 4 && numbers.every(Number.isFinite) ? numbers : [0.22, 0.61, 0.36, 1];
};

const points = computed(() => parseCurve(props.modelValue));
const format = (value) => Number(value.toFixed(2)).toString();
const update = (index, value) => {
	const next = [...points.value];
	next[index] = value;
	emit("update:modelValue", `cubic-bezier(${next.map(format).join(", ")})`);
};

const controls = [
	{ index: 0, label: "X1", min: 0, max: 1 },
	{ index: 1, label: "Y1", min: -1, max: 2 },
	{ index: 2, label: "X2", min: 0, max: 1 },
	{ index: 3, label: "Y2", min: -1, max: 2 },
];

const pointX = (index) => 12 + points.value[index] * 136;
const pointY = (index) => 88 - points.value[index] * 76;
const curve = computed(() => `M12 88 C${pointX(0)} ${pointY(1)}, ${pointX(2)} ${pointY(3)}, 148 12`);
const handles = computed(() => `M12 88 L${pointX(0)} ${pointY(1)} M148 12 L${pointX(2)} ${pointY(3)}`);
</script>

<style scoped>
.thin-slider :deep([data-orientation="horizontal"]) {
	height: 2px;
}

.thin-slider :deep([role="slider"]) {
	height: 10px;
	width: 10px;
}
</style>
