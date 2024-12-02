<script lang="ts">
	import Dropdown from '$lib/components/Dropdown.svelte';
	import Label from '$lib/components/Label.svelte';
	import { colors, getAccentColor, setAccentColor } from '$lib/theme';
	import { capitalize } from '$lib/util';

	let value = getAccentColor();
	const colorslocal = [
        { value: "red", label: "红色" },
		{ value: "orange", label: "橙色" },
		{ value: "amber", label: "琥珀色" },
		{ value: "yellow", label: "黄色" },
		{ value: "lime", label: "青柠色" },
		{ value: "green", label: "绿色" },
		{ value: "emerald", label: "祖母绿色" },
		{ value: "teal", label: "青色" },
		{ value: "cyan", label: "蓝绿色" },
		{ value: "sky", label: "天蓝色" },
		{ value: "blue", label: "蓝色" },
		{ value: "indigo", label: "靛蓝色" },
		{ value: "violet", label: "紫罗兰色" },
		{ value: "purple", label: "紫色" },
		{ value: "fuchsia", label: "紫红色" },
		{ value: "pink", label: "粉色" },
		{ value: "rose", label: "玫瑰色" }
	];
</script>

<div class="flex items-center">
	<Label text="主题色"
		>更改高亮元素（如按钮和复选框）的颜色。</Label
	>

	<Dropdown
		class="flex-grow"
		selected={colorslocal.find(item => item.value === value)|| colorslocal[0]}
		items={colorslocal}
        getLabel={(item) => item.label}
		onSelectedChange={(newValue) => {
			value = newValue.value;
			setAccentColor(value);
		}}
		multiple={false}
	/>
</div>
