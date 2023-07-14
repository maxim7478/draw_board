<template>
  <div class="board-toolbar">
      <div class="board-toolbar__wrapper">
				<div
					v-for="item in tools"

					:key="`iconTool${item.id}`"
					@click="changeTool(item.name)"
					class="board-toolbar__item"
					:class="{'board-toolbar__item-active': currentTool === item.name}"
				>
					<component
						:is="item.icon"
					/>
					<span>{{ item.label }}</span>
				</div>
        <input v-model="currentFillFigure" type="checkbox" class="board-toolbar__item board-toolbar__checkbox">
				<div class="board-toolbar__input-wr">
					<input v-model="currentFontSize" type="range" min="1" max=120 class="">
					<span>Шрифт: {{currentFontSize}}</span>
				</div>
        <input v-model="currentActiveColor" type="color" class="board-toolbar__item board-toolbar__selector-color">
				<div class="board-toolbar__input-wr">
					<input v-model="currentLineSize" type="range" min="1" max="30" class="board-toolbar__slider">
					<span>Размер: {{currentLineSize}}</span>
				</div>
				<div class="board-toolbar__back-btn">
					<icon-back-arrow @click="emit('backCanvas')"/>
					<icon-forward @click="emit('forwardCanvas')"/>
				</div>
				<button @click="emit('clearCanvas')" class="board-toolbar__button">Clear</button>
				<button @click="emit('saveCanvas')" class="board-toolbar__button">Save</button>
      </div>
  </div>
</template>

<script setup lang="ts">
import {computed, ref} from "vue";
import IconTreg from "@/components/icons/IconTreg.vue";
import IconRect from "@/components/icons/IconRect.vue";
import IconCircle from "@/components/icons/IconCircle.vue";
import IconPencil from "@/components/icons/IconPencil.vue";
import IconLine from "@/components/icons/IconLine.vue";
import TextIcon from "@/components/icons/text-icon.vue";
import IconClear from "@/components/icons/IconClear.vue";
import IconCursor from "@/components/icons/IconCursor.vue";
import IconBackArrow from "@/components/icons/IconBackArrow.vue";
import IconForward from "@/components/icons/IconForward.vue";

interface IProps {
	activeColor: string;
	activeTool: string;
	lineSize: number;
	fontSize: number;
	fillFigure: boolean;
}
interface IEmit {
	(event: "changeToolParam", value: { type: string; value: string })
	(event: "clearCanvas"): void
	(event: "saveCanvas"): void
	(event: "backCanvas"): void
	(event: "forwardCanvas"): void
}

const props = defineProps<IProps>();
const emit = defineEmits<IEmit>();


const tools = ref([
	{
		id: 0,
		name: "cursor",
		active: false,
		icon: IconCursor,
		label: "Курсор"
	},
	{
		id: 1,
		name: "rectangle",
		active: false,
		icon: IconRect,
		label: "Квадрат"
	},
	{
		id: 2,
		name: "triangle",
		icon: IconTreg,
		label: "Треугольник"
	},
	{
		id: 3,
		name: "circle",
		icon: IconCircle,
		label: "Круг"
	},
	{
		id: 4,
		name: "line",
		icon: IconLine,
		label: "Линия"
	},
	{
		id: 5,
		name: "pencil",
		icon: IconPencil,
		label: "Перо"
	},
	{
		id: 6,
		name: "clear",
		icon: IconClear,
		label: "Ластик"
	},
	{
		id: 7,
		name: "text",
		icon: TextIcon,
		label: "Текст",
	},
])

const changeTool = (itemName) => {
	emit("changeToolParam", {type: "tool", value: itemName})
}

const currentTool = computed(() => {
	return props.activeTool;
})



const currentActiveColor = computed({
	get: () => {
		return props.activeColor
	},
	set: (value) => {
		emit("changeToolParam", { type: "color", value })
	}
})

const currentLineSize = computed({
	get: () => {
		return props.lineSize
	},
	set: (value) => {
		emit("changeToolParam", { type: "lineSize", value })
	}
})

const currentFontSize = computed({
	get: () => {
		return props.fontSize
	},
	set: (value) => {
		emit("changeToolParam", { type: "fontSize", value })
	}
})

const currentFillFigure = computed({
	get: () => {
		return props.fillFigure
	},
	set: (value) => {
		emit("changeToolParam", { type: "fillFigure", value })
	}
})
</script>

<style scoped lang="scss">
.board-toolbar {
  height: max-content;
	background-color: #e0eefd;
	width: max-content;
	border-right: 1px solid black;
	border-top: 1px solid black;
	border-bottom: 1px solid black;
	border-bottom-right-radius: 20px;
	border-top-right-radius: 20px;
  &__wrapper {
    padding: 1rem;
		display: flex;
		flex-flow: column;
		align-items: start;
		width: max-content;
  }
	&__input-wr {
		display: flex;
		flex-flow: column;
		align-items: center;
		width: auto;
		& input {
			min-width: 100px;
		}
	}
	&__item {
		margin: 0.7rem 0.3rem;
		cursor: pointer;
		fill: black;
		color: black;
		display: flex;
		align-items: center;
		gap: 1rem;
		&-active {
			color: blue;
			fill: blue;
		}
		& svg {
			width: 30px;
			height: 30px;
		}
	}
	&__selector-color {
		-webkit-appearance: none;
		border: 1px solid black;
		border-radius: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		width: 30px;
		height: 30px;
	}
	&__slider {
		min-width: 100px;
	}
	&__font {
		min-height: 30px;
		width: 50px;
	}
	&__checkbox {
		min-height: 20px;
		width: 50px;
	}
	&__button {
		margin: 0.7rem 0.3rem;
	}
	input[type="color"]::-webkit-color-swatch-wrapper {
		padding: 0;
		width: 20px;
		height: 20px;
		margin: 0;
	}
	input[type="color"]::-webkit-color-swatch {
		border: none;
		border-radius: 100%;
	}
	&__back-btn {
		display: flex;
		gap: 1rem;
		fill: black;
		color: black;
		justify-content: center;
		width: 100%;
		& svg {
			height: 30px;
			width: 30px;
			cursor:  pointer;
		}
	}
}
</style>
