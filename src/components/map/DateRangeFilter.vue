<template>
	<div id="dateRangeFilter">
		<NcDateTimePicker
			type="date-range"
			:model-value="pickerValue"
			:min="minDate"
			:max="maxDate"
			:clearable="false"
			:append-to-body="true"
			:aria-label="t('maps', 'Time filter date range')"
			@update:model-value="onPickerChange" />
		<NcButton
			:aria-label="t('maps', 'Show all dates')"
			:title="t('maps', 'Show all dates')"
			@click="onReset">
			{{ t('maps', 'Reset') }}
		</NcButton>
	</div>
</template>

<script>
import { NcButton, NcDateTimePicker } from '@nextcloud/vue'

export default {
	name: 'DateRangeFilter',

	components: {
		NcButton,
		NcDateTimePicker,
	},

	props: {
		start: {
			type: Number,
			required: true,
		},
		end: {
			type: Number,
			required: true,
		},
		rangeMin: {
			type: Number,
			required: true,
		},
		rangeMax: {
			type: Number,
			required: true,
		},
	},

	emits: ['range-change'],

	computed: {
		// first selectable day (dates before are grayed out)
		minDate() {
			const d = new Date(this.rangeMin * 1000)
			d.setHours(0, 0, 0, 0)
			return d
		},
		// last selectable day (dates after are grayed out)
		maxDate() {
			const d = new Date(this.rangeMax * 1000)
			d.setHours(23, 59, 59, 999)
			return d
		},
		pickerValue() {
			// the filter can be wider than the data (e.g. 0 when "everything" is selected), keep display inside the data range
			const start = Math.min(Math.max(this.start, this.rangeMin), this.rangeMax)
			const end = Math.max(Math.min(this.end, this.rangeMax), start)
			return [new Date(start * 1000), new Date(end * 1000)]
		},
	},

	methods: {
		onPickerChange(dates) {
			if (!Array.isArray(dates) || !dates[0] || !dates[1]) {
				return
			}
			// whole days, in local time: first day 00:00:00 to last day 23:59:59
			const startDate = new Date(dates[0])
			startDate.setHours(0, 0, 0, 0)
			const endDate = new Date(dates[1])
			endDate.setHours(23, 59, 59, 999)
			this.$emit('range-change', {
				start: Math.floor(startDate.getTime() / 1000),
				end: Math.floor(endDate.getTime() / 1000),
			})
		},
		onReset() {
			this.$emit('range-change', { start: this.rangeMin, end: this.rangeMax })
		},
	},
}
</script>

<style lang="scss" scoped>
#dateRangeFilter {
	position: absolute;
	left: 50%;
	bottom: 30px;
	transform: translateX(-50%);
	z-index: 10000;
	display: flex;
	align-items: center;
	gap: 8px;
	padding: 6px;
	border-radius: var(--border-radius-large);
	background-color: var(--color-main-background);
	box-shadow: 0 0 10px var(--color-box-shadow);
}
</style>
