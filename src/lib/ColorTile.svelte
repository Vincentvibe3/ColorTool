<script lang="ts">
	import { fly, type FlyParams } from "svelte/transition";
  	import type { ColorData } from "./ColorData";
  	import { contrast, HEX2HSV, HEX2RGB, HSV2HEX } from "./colorUtils";
  import { createEventDispatcher } from "svelte";
	export let row:number;
	export let column:number;
	export let color:ColorData={
		color:"#ffffff",
		label:`${row} ${column}`
	};
	export let bgColor:ColorData={
		color:"#ffffff",
		label:`${row} ${column}`
	};

	export let group:ColorData;
	export let contrastValue=0; 

	const dispatch = createEventDispatcher();

	$: contrastValue=contrast(HEX2RGB(color.color as string), HEX2RGB(bgColor.color as string))

	let textColor = "#ffffff"
	let borderColor = "#ffffff"
	let dark = false

	$: if (contrast(HEX2RGB(color.color as string), {r:255, g:255, b:255})>contrast(HEX2RGB(color.color as string), {r:0, g:0, b:0})){
		textColor = "#ffffff"
		let hsv = HEX2HSV(color.color)
		hsv.v=hsv.v+0.3
		borderColor=HSV2HEX(hsv)
		dark = true
	} else {
		textColor = "#000000"
		let hsv = HEX2HSV(color.color)
		hsv.v=hsv.v-0.3
		dark = false
		borderColor=HSV2HEX(hsv)
	}

	let checked: boolean

	$: checked = group==color
	
	export let direction:string;
	let flyParams:FlyParams;

	$: if (direction=="row"){
		flyParams = {duration:200, y:10}
	} else {
		flyParams = {duration:200, x:-10}
	}

	let checkbox:HTMLInputElement;

	const onClick = ()=>{
		checkbox.click()
		dispatch("click");
	}

</script>
<div
in:fly="{flyParams}"
out:fly="{flyParams}"
>
	<input 
	class="checkbox"
	type="radio"
	bind:group={group}
	bind:this={checkbox}
	value={color}>
	<button 
	on:click={onClick}
	style:background-color={color.color} 
	style:color={textColor}
	style:outline-color={borderColor}
	class:selected={checked}
	class:colorChip={true} class:dark class:light={!dark}>
		{Math.round(contrastValue*100)/100}
	</button>
</div>
<style>
	.checkbox{
		display: none;
	}
	
	.colorChip{
		aspect-ratio: 1;
		width: 4rem;
		outline-width: 0.1rem;
		outline: solid 0.15rem;
		border: none;
		border-radius: 0.5rem;
		margin: 0.5rem;
		transition: all 0.2s ease-in-out;
		font: var(--body);
	}

	.colorChip.selected{
		outline-width: 0.25rem;
	}

	.colorChip.light:hover {
		filter: brightness(0.8);
	}

	.colorChip.dark:hover {
		filter: brightness(1.2);
	}

	.colorChip.light:active {
		filter: brightness(0.6);
	}

	.colorChip.dark:active {
		filter: brightness(1.4);
	}
</style>