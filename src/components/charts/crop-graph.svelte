<script lang="ts">
	import { toReadable } from '$lib/format';
	import { scaleLinear } from 'd3-scale';
	import { extent } from 'd3-array';
	import { Crop, CROP_TO_PEST, getCropDisplayName, getCropFromName } from 'farming-weight';
	import { Area, Axis, Chart, Highlight, Spline, Svg, Tooltip } from 'layerchart';

	interface Props {
		data: { date: string; value: number; pests: number }[];
		pests?: boolean;
		ratio?: number;
		crop?: string;
	}

	let { data, pests = false, ratio = 0, crop = 'wheat' }: Props = $props();

	let first = $derived(data[0]);
	let last = $derived(data.at(-1));
	let yDomain = $derived.by(() => {
		const domain = last ? [first.value, last.value + (last.value - first.value) * (1 - ratio)] : undefined;
		if (!domain || first.value === last?.value) {
			return undefined;
		}
		return domain;
	});
	let days = $derived(Math.ceil((+(last?.date ?? 0) - +first.date) / 86400));

	const dateFormatter = new Intl.DateTimeFormat(undefined, {
		month: 'short',
		day: 'numeric',
	});

	const tooltipFormatter = new Intl.DateTimeFormat(undefined, {
		month: 'short',
		day: '2-digit',
		hour: '2-digit',
		minute: '2-digit',
		year: 'numeric',
	});

	let pestScale = $derived(scaleLinear(extent(data, (d) => d.pests) as [number, number], yDomain ?? [0, 1]));

	const colorClasses: Record<string, string[]> = {
		wheat: ['stroke-wheat', 'fill-wheat/70', 'stroke-primary/80'],
		melon: ['stroke-melon', 'fill-melon/70', 'stroke-primary/80'],
		cactus: ['stroke-cactus', 'fill-cactus/70', 'stroke-primary/80'],
		pumpkin: ['stroke-pumpkin', 'fill-pumpkin/70', 'stroke-primary/80'],
		carrot: ['stroke-carrot', 'fill-carrot/70', 'stroke-primary/80'],
		potato: ['stroke-potato', 'fill-potato/70', 'stroke-primary/80'],
		cane: ['stroke-sugarcane', 'fill-sugarcane/70', 'stroke-primary/80'],
		wart: ['stroke-netherwart', 'fill-netherwart/70', 'stroke-primary/80'],
		mushroom: ['stroke-mushroom', 'fill-mushroom/70', 'stroke-primary/80'],
		cocoa: ['stroke-cocoa', 'fill-cocoa/70', 'stroke-primary/80'],
	};
</script>

<div class="m-2 h-[200px] rounded">
	<Chart
		{data}
		x="date"
		y="value"
		{yDomain}
		yNice
		padding={{ left: 48, bottom: 16, top: 5, right: 48 }}
		tooltip={{ mode: 'bisect-x' }}
		let:height
	>
		<Svg>
			<Axis
				placement="left"
				rule
				grid
				format={(d: number) => toReadable(d)}
				tickLabelProps={{ class: '!stroke-0 !font-normal text-sm' }}
			/>
			<Axis
				placement="bottom"
				rule
				format={(d: number) => dateFormatter.format(new Date(d * 1000))}
				tickLabelProps={{
					rotate: 330,
					textAnchor: 'end',
					class: '!stroke-0 !font-normal text-xs md:text-sm',
				}}
				ticks={days > 14 ? 14 : days}
			/>
			{#if pests}
				<Axis
					placement="right"
					scale={scaleLinear(pestScale.domain(), [height, 0])}
					ticks={pestScale.ticks()}
					format={(v: number) => toReadable(v, undefined, 2)}
					rule
					tickLabelProps={{ class: '!stroke-0 !font-normal text-sm' }}
				/>
				<Spline class="{colorClasses[crop][0]} stroke-[3]" />
				<Spline class="{colorClasses[crop][2]} stroke-[3]" y={(d: { pests: number }) => pestScale(d.pests)} />
				<Highlight points={{ class: 'fill-primary' }} y={(d: { pests: number }) => pestScale(d.pests)} />
			{:else}
				<Area line={{ class: colorClasses[crop][0] + ' stroke-4' }} class={colorClasses[crop][1]} />
				<Spline class="{colorClasses[crop][0]} stroke-2" />
			{/if}
			<Highlight lines points={{ class: colorClasses[crop][0] }} />
		</Svg>
		<Tooltip.Root let:data>
			<Tooltip.Header>
				{tooltipFormatter.format(new Date(data.date * 1000))}
			</Tooltip.Header>
			<div class="min-w-lg">
				<p>{getCropDisplayName(getCropFromName(crop) ?? Crop.Wheat)} Collection</p>
				<p class="font-mono">{(+data.value).toLocaleString()}</p>
				{#if pests}
					<p class="first-letter:capitalize">{CROP_TO_PEST[getCropFromName(crop) ?? Crop.Wheat]} Kills</p>
					<p class="font-mono">{(+data.pests).toLocaleString()}</p>
				{/if}
			</div>
		</Tooltip.Root>
	</Chart>
</div>
