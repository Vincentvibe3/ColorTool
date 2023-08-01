<script lang="ts">
	import { browser } from "$app/environment";
  	import type { ColorData } from "$lib/ColorData";
  	import ColorTile from "$lib/ColorTile.svelte";
	import { onMount } from "svelte";
	import { compressToEncodedURIComponent, decompressFromEncodedURIComponent } from "lz-string"
	import {Button, ContentContainer, TextInput, ColorPicker} from "nota-ui"
  	import { HEX2RGB, contrast } from "$lib/colorUtils";
	let tempColor = "#ffffff"
	export let currentColor:ColorData={
		color:"#ffffff",
		label:"label"
	};
	let columns = 5;
	let rows = 2;
	let colors:any=[];
	let mounted=false
	let textColor: string
	let direction:string;

	$: for (let row in [...Array(rows+1).keys()]){
		colors.push([])
	}

	const tileClicked = () => {
		tempColor = currentColor.color
		colors = colors
		onColorInputChange()
	}

	const onColorInputChange = () => {
		// Force update of all colors
		colors = colors
		// Force update of color display
		currentColor = currentColor
		save()
	}

	$: if (mounted){
		currentColor.color = tempColor
		onColorInputChange()
	}

	const resetColors = ()=>{
		colors.forEach((row: ColorData[]) => {
			row.forEach( (column: ColorData) => {
				column.color = "#FFFFFF"
			})
		});
		onColorInputChange()
	}

	const load = ()=>{
		var searchParams = new URLSearchParams(window.location.search)
		let b64data = searchParams.get("data")
		if (b64data!=null){
			var data = decodeURIComponent(escape(window.atob(b64data))).split(".")
			console.log(data)
			rows = Number.parseInt(data[0])
			columns = Number.parseInt(data[1])
			console.log(atob(b64data).split("."))
			console.log(rows)
			console.log(columns)
			for (let index=2;index<data.length;index++){
				let entry = data[index].split(":")
				let color = entry[0]
				let row = entry[1]
				
				let column = entry[2]
				if (colors[row]===undefined){
					colors.push([])
				}
				if (colors[row][column]===undefined){
					colors[row][column] = {
						color:`#${color}`,
						label:`${row} ${column}`
					}
				} else {
					colors[row][column].color = `#${color}`
				}
			}
		}
	}

	const save = ()=>{
		console.log(colors)
		console.log(rows)
		console.log(columns)
		if (mounted){
			let data = [
				rows,
				columns,
				`${colors[0][0].color.replace("#" ,"")}:0:0`,
				`${colors[0][1].color.replace("#" ,"")}:0:1`
			]
			for(let row in [...Array(rows+1).keys()]){
				if (row=="0"){
					continue
				}
				for (let column in [...Array(columns).keys()]){
					console.log(column)
					data.push(`${colors[row][column].color.replace("#" ,"")}:${row}:${column}`)
				}
			}
			let dataToWrite = window.btoa(unescape(encodeURIComponent(data.join("."))))
			history.pushState({}, "", `${window.location.origin}?data=${dataToWrite}`)
		}
		
	}

	$: if (mounted&&contrast(HEX2RGB(colors[0][0].color), {r:255, g:255, b:255})>contrast(HEX2RGB(colors[0][0].color), {r:0, g:0, b:0})){
		textColor = "#ffffff"
	} else {
		textColor = "#000000"
	}

	onMount(()=>{
		load()
		currentColor=colors[0][0]
		tempColor = currentColor.color
		mounted = true
	})

</script>
<ContentContainer>
	<div class="controls">
		<p>Controls</p>
		<ContentContainer --contentContainerPaddingy="4rem" --contentContainerPaddingx="1rem" alignment="flex-start">
			<!-- <ContentContainer> -->
				<ColorPicker bind:text={tempColor}></ColorPicker>
				<!-- <div style="flex-grow:1;">
					<TextInput on:change={onColorInputChange} on:enterPressed={onColorInputChange} captureEnter bind:text={currentColor.color}/>
				</div>
				<input on:change={onColorInputChange} bind:value={currentColor.color} type="color"> -->
			<!-- </ContentContainer> -->
			<ContentContainer direction="row">
				<Button on:click={()=>{direction="row";rows++;onColorInputChange()}}>
					Add Row
				</Button>
				<Button on:click={()=>{if (rows > 0){ direction="row";rows--;onColorInputChange()}}}>
					Remove Row
				</Button>
				<Button on:click={()=>{direction="column";columns++;onColorInputChange()}}>
					Add Column
				</Button>
				<Button on:click={()=>{if (columns > 0){ direction="column";columns--;onColorInputChange()}}}>
					Remove Column
				</Button>
				<Button on:click={resetColors}>
					Reset Colors
				</Button>
			</ContentContainer>
			
		</ContentContainer>
	</div>
	
	<ContentContainer direction="column">
		<div class="bg" style:background-color={colors[0][0]?.color}>
			<!-- <p>{currentColor.color}</p> -->
			<!-- <p>{currentColor.label}</p> -->
			<ContentContainer direction="column" alignment="flex-start">
				<div>
					<p style:color={textColor}>Bg/Fg</p>
					<div class="row">
						<ColorTile
						on:click={tileClicked}
						bind:bgColor={colors[0][0]}
						bind:direction={direction}
						row={0}
						column={0}
						bind:group={currentColor}  bind:color={colors[0][0]}>
						</ColorTile>
						<ColorTile
						on:click={tileClicked}
						bind:bgColor={colors[0][0]}
						bind:direction={direction}
						row={0}
						column={1}
						bind:group={currentColor} bind:color={colors[0][1]}>
						</ColorTile>
					</div>
					<p style:color={textColor}>Accents</p>
					{#each  [...Array(rows).keys()]  as rowsIndex}
						<div class="row">
							{#each [...Array(columns).keys()] as columnIndex}
								<ColorTile
								on:click={tileClicked}
								bind:bgColor={colors[0][0]}
								bind:direction={direction}
								row={rowsIndex+1}
								column={columnIndex}
								bind:group={currentColor} 
								bind:color={colors[(rowsIndex+1)][columnIndex]}>
								</ColorTile>
							{/each}
						</div>
					{/each}
				</div>
			</ContentContainer>
			
		</div>
	</ContentContainer>
</ContentContainer>




<style>
	
	.row {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: start;
	}

	.bg {
		display: flex;
		min-width: 100%;
		width: fit-content;
		flex-grow: 1;
		padding: 2rem;
		padding-top: 4rem;
		min-height: 100vh;
	}

	@media only screen and (min-width: 1000px) {
		.bg {
			/* align-items: center; */
			/* justify-content: center; */
		}
	}

	.controls {
		width: 100%;
	}

	@media only screen and (min-width: 1000px) {
		.controls {
			width: 30%;
			align-self: flex-start;
		}
	}
</style>