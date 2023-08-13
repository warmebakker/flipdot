<script setup lang="ts">
import { reactive, ref } from 'vue';

// flipDot module size
const moduleWidth: number = 5;
const moduleHeight: number = 7;

class Flip {
	active: boolean;
	constructor(active: boolean) {
		this.active = active;
	}
}

class Arrangement {
	Name: string;
	horizontal: number;
	vertical: number;

	constructor(Name: string, horizontal: number, vertical: number) {
		this.Name = Name;
		this.horizontal = horizontal;
		this.vertical = vertical;
	}
}

class SelectedArrangement {
	modulesHorizontal: number = 1;
	modulesVertical: number = 1;

	get modulesTotal(): number { 
		return this.modulesHorizontal * this.modulesVertical;
	}
	get columns(): number { 
		return this.modulesHorizontal * moduleWidth;
	}
	get rows(): number { 
		return this.modulesVertical * moduleHeight;
	}
	get pixels(): number { 
		return this.columns * this.rows;
	}

	updateArrangement(modulesHorizontal: number, modulesVertical: number) {
		this.modulesHorizontal = modulesHorizontal;
		this.modulesVertical = modulesVertical;
	}
}

class Tool {
	tool: string = 'paint';
	paints: boolean = true;
	
	get erases(): boolean {
		return !this.paints;
	}	
}

const selectedArrangement = reactive<SelectedArrangement>(new SelectedArrangement());

const arrangements = ref<Arrangement[]>([
	new Arrangement('1x1 (Hor x Ver)',1,1), 
	new Arrangement('1x2',1,2),
	new Arrangement('1x3',1,3),
	new Arrangement('1x4',1,4),
	new Arrangement('2x1 (Hor x Ver)',2,1), 
	new Arrangement('2x2',2,2),
	new Arrangement('2x3',2,3),
	new Arrangement('2x4',2,4),
	new Arrangement('3x1',3,1),
	new Arrangement('3x2',3,2),
	new Arrangement('3x3',3,3)
]);

const paintTool = ref<Tool>(new Tool());

const pixels = reactive<Flip[]>(Array.from(
	{ length: selectedArrangement.pixels }, 
	() => (new Flip(false) )));

const updateArrangement = (e: any) => {
	const selected = arrangements.value.find(a => a.Name === e.target.value) || arrangements.value[0];
	selectedArrangement.updateArrangement(selected.horizontal, selected.vertical);
	if(pixels.length === selectedArrangement.pixels) return;

	if(pixels.length > selectedArrangement.pixels) 
	pixels.splice(selectedArrangement.pixels, pixels.length - selectedArrangement.pixels);
	else 
	pixels.push(...Array.from(
			{ length: selectedArrangement.pixels - pixels.length }, 
			() => (new Flip(false) )));
}
function paint(e: any, b: Flip) {
	if(e.buttons == 0) return;
	e.preventDefault();	
	b.active = paintTool.value.paints;
}
</script>

<template>
	<div>
		<select name="arrangements" id="modArr" @change="updateArrangement">
			<option v-for="a in arrangements" :key="a.Name" :value="a.Name">{{ a.Name }}</option>
		</select>
		<button @click="pixels.forEach(b => b.active = Math.floor(Math.random()* 2) == 0)">Random</button>
		<button @click="pixels.forEach(b => b.active = false)">Reset all</button>
		<button @click="pixels.forEach(b => b.active = true)">Set all</button>
		<button @click="pixels.forEach(b => b.active = !b.active)">Invert</button>
		<label class="light" for="painter">Paint</label>
		<input type="radio" id="painter" name="tool" value="Paint" :checked="paintTool.paints" @change="paintTool.paints=true">
		<label class="light" for="eraser">Erase</label>
		<input type="radio" id="eraser" name="tool" value="Erase" @change="paintTool.paints=false">
	</div>
	<div class="container">
		<div class="gridWrap" :style="{ gridTemplateColumns: 'repeat(' + selectedArrangement.columns + ', auto)'}">
			<div v-for="b in pixels" 
				@click="b.active = !b.active"
				@mouseenter="(event) => paint(event, b)"
				class="gridCell" :class="{ activated: b.active }"></div>
		</div>
	</div>
</template>

<style >
body {
	margin: 0;
}
#app {
	position: fixed;
	height: 100%;
	width: 100%;
	background: #000;
}
.light {
	color: #fff;
}
.container {
	height: 100%;
	display: flex;
	justify-content: center;
	align-items: center;
	margin: 2rem;
}
.gridWrap {
	display: grid;
	background: #333;
	margin-top: -10rem;
}
.gridCell {
	cursor: pointer;
	width: 30px;
	height: 30px;
	border: 1px solid #999;
	margin: .1rem; 
	transition: all 0.15s ease-out;
	border-radius: .75rem;
}
.activated {
	background-color: rgb(194, 210, 91);
}
</style>
