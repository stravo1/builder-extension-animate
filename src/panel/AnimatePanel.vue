<template>
	<div class="flex h-full flex-col gap-4 overflow-y-auto bg-surface text-ink-gray-8">
		<p v-if="!blockId" class="pt-2 text-p-sm italic text-ink-gray-5">Select a block to animate it.</p>

		<Tabs v-else v-model="activeTab" :tabs="tabs">
			<template #tab-panel="{ tab }">
				<section v-if="tab.label === 'Effects'" class="flex flex-col gap-3 p-3">
					<AnimationPreview
						:effect="settings.effect"
						:duration="settings.duration"
						:delay="settings.delay"
						:easing="settings.easing" />
					<EffectGrid v-model="settings.effect" />
					<Button variant="subtle" size="sm" label="Remove animation" @click="settings.effect = 'none'" />
				</section>

				<section v-else-if="tab.label === 'Timing'" class="flex flex-col gap-4 p-3">
					<template v-if="settings.effect !== 'none'">
						<div class="grid grid-cols-2 gap-1.5">
							<Button
								v-for="option in TRIGGERS"
								:key="option.value"
								:title="option.hint"
								size="sm"
								:variant="option.value === settings.trigger ? 'subtle' : 'ghost'"
								class="w-full"
								:label="option.label"
								@click="settings.trigger = option.value" />
						</div>
						<SliderRow v-model="settings.duration" label="Duration" :min="100" :max="2000" :step="50" />
						<SliderRow v-model="settings.delay" label="Delay" :min="0" :max="1500" :step="50" />
						<div class="flex items-center justify-between gap-2">
							<span class="text-p-sm text-ink-gray-6">Repeat</span>
							<Select v-model="settings.repeat" :options="REPEATS" class="w-32" size="sm" />
						</div>
					</template>
					<p v-else class="text-p-sm text-ink-gray-5">Choose an effect before setting its timing.</p>
				</section>

				<section v-else class="flex flex-col gap-4 p-3">
					<template v-if="settings.effect !== 'none'">
						<div class="flex items-center justify-between gap-2">
							<span class="text-p-sm text-ink-gray-6">Easing</span>
							<Select
								v-model="easingMode"
								:options="[...EASINGS, { label: 'Custom curve', value: 'custom' }]"
								class="w-32"
								size="sm" />
						</div>
						<BezierEditor v-model="settings.easing" />
					</template>
					<p v-else class="text-p-sm text-ink-gray-5">Choose an effect before setting its easing.</p>
				</section>
			</template>
		</Tabs>
	</div>
</template>

<script setup>
/**
 * The popover: every animation setting for the selected block, with a preview
 * the editor canvas cannot give.
 *
 * The settings object is the one owner while a block stays selected. It is
 * filled from the block when the selection moves, and written back on every
 * change. Reading the block after each write would race the editor and undo the
 * user's next keystroke.
 */
import builder from "frappe-builder-extension-sdk";
import { useBuilderContext } from "frappe-builder-extension-sdk/vue";
import { Button, Select, Tabs } from "frappe-ui";
import { computed, reactive, ref, watch } from "vue";
import { DEFAULTS, EASINGS, REPEATS, TRIGGERS, readSettings, toAttributes } from "../animation.js";
import { EASING_VALUES } from "../runtime.js";
import AnimationPreview from "./AnimationPreview.vue";
import BezierEditor from "./BezierEditor.vue";
import EffectGrid from "./EffectGrid.vue";
import SliderRow from "./SliderRow.vue";

const context = useBuilderContext(["selection", "readOnly"]);
const blockId = computed(() => context.selection.blockId);

const settings = reactive({ ...DEFAULTS });
const tabs = [
	{ label: "Effects", icon: "lucide-sparkles" },
	{ label: "Timing", icon: "lucide-timer" },
	{ label: "Easing", icon: "lucide-chart-spline" },
];
const activeTab = ref(0);
const easingMode = computed({
	get: () => (EASINGS.some((option) => option.value === settings.easing) ? settings.easing : "custom"),
	set: (value) => {
		settings.easing = value === "custom" ? EASING_VALUES[settings.easing] || settings.easing : value;
	},
});
/** True while the block is being read, so filling the form does not write it back. */
const loading = ref(false);

const load = async (id) => {
	loading.value = true;
	try {
		const block = id ? await builder.block.get(id) : null;
		Object.assign(settings, readSettings(block?.attributes));
	} finally {
		loading.value = false;
	}
};

watch(blockId, load, { immediate: true });

watch(
	() => ({ ...settings }),
	async (next) => {
		if (loading.value || !blockId.value || context.readOnly) return;

		await builder.block.update(blockId.value, { attributes: toAttributes(next) });
		// the entry frame owns the runtime, and knows whether this page has it
		if (next.effect !== DEFAULTS.effect) await builder.actions.run("animate.touched");
	},
);
</script>
