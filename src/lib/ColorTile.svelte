<script lang="ts">
	import { fly, type FlyParams } from "svelte/transition";
  	import type { ColorData } from "./ColorData";
  	import { contrast, HEX2HSV, HEX2RGB, HSV2HEX } from "./colorUtils";
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

	$: contrastValue=contrast(HEX2RGB(color.color as string), HEX2RGB(bgColor.color as string))

	let textColor = "#ffffff"
	let borderColor = "#ffffff"

	$: if (contrast(HEX2RGB(color.color as string), {r:255, g:255, b:255})>contrast(HEX2RGB(color.color as string), {r:0, g:0, b:0})){
		textColor = "#ffffff"
		let hsv = HEX2HSV(color.color)
		hsv.v=hsv.v+0.3
		borderColor=HSV2HEX(hsv)
	} else {
		textColor = "#000000"
		let hsv = HEX2HSV(color.color)
		hsv.v=hsv.v-0.3
		borderColor=HSV2HEX(hsv)
	}
	
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
	class="colorChip">
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
	}
</style>