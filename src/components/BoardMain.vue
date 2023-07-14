<template>
  <div class="board-main">
    <BoardToolbar
			:active-color="activeColor"
			:active-tool="activeTool"
			:line-size="lineSize"
			:font-size="fontSize"
			:fill-figure="fillFigure"
			@changeToolParam="changeBoardTool"
			@clearCanvas="clearCanvas"
			@saveCanvas="saveCanvas"
			@backCanvas="onBackCanvas"
			@forwardCanvas="onForwardCanvas"
		/>
    <div ref="canvasWrapperRef" class="board-main__wrapper"
				 @mouseup="onStopDraw">
<!--			<div-->
<!--				v-if="activeTool === 'text' && currentAction === 'writing'"-->
<!--				style="width: max-content; height: max-content"-->
<!--				:style="`position: fixed; width: max-content; height: max-content; left: ${currentPositionX + 200}px; top: ${currentPositionY}px`"-->
<!--			>-->
<!--				<textarea-->
<!--					ref="textAreaRef"-->
<!--					:style="`color: ${activeColor}; font-size: ${fontSize}px`"-->
<!--				/>-->
<!--				<div-->
<!--					@mousedown="isSelect = true"-->
<!--					@mouseup="isSelect = false"-->
<!--					style="height: 10px; width: 10px; border-radius: 100%; background: blue; position: absolute; top: -15px; left: -15px; cursor: pointer"-->
<!--				/>-->
<!--			</div>-->
      <canvas
				id="canvas"
				:class="[`current-cursor-${activeTool}`]"
				@click="onClickCanvasElement"
				@mousemove="onDraw"
				@mousedown="onStartDraw"
				@mouseup="onStopDraw"
				@keydown.enter="onUpdateElementData"
				@dblclick="saveElement"
			/>
    </div>
  </div>
</template>

<script setup lang="ts">
import BoardToolbar from "@/components/BoardToolbar.vue";
import {onMounted, ref, watch} from "vue";

const ctx = ref<CanvasCompositing>();
const canvasWrapperRef = ref(null)
const currentCanvasIndex = ref<number | null>(null)
const canvasRef = ref<HTMLCanvasElement>();
const historyCach = ref([]);
const isDrawing = ref(false);
const textAreaRef = ref<HTMLElement>();
const isSelect = ref(false);
const currentAction = ref("");
const isFocusTextarea = ref(false);
const listTextElements = ref([])
const isDrawAction = ref(false);
const snapshot = ref(false);
const prevMouseX = ref();
const currentPositionX = ref();
const prevMouseY = ref();
const currentPositionY = ref();
const activeColor = ref("#000");
const activeTool = ref("cursor");
const lineSize = ref(16);
const fontSize = ref(12);
const fillFigure = ref(false);

onMounted(() => {
	canvasRef.value = document.getElementById("canvas") as HTMLCanvasElement
	canvasRef.value.width = canvasRef.value?.offsetWidth
	canvasRef.value.height = canvasRef.value?.offsetHeight
	ctx.value = canvasRef.value.getContext("2d")
	setCanvasBackground()
	// window.addEventListener("paste", onPasteElement)
})

const onStartDraw = ($event) => {
	if (currentAction.value === "writing") return
	currentPositionY.value = $event.offsetY
	currentPositionX.value = $event.offsetX
	prevMouseX.value = $event.offsetX
	prevMouseY.value = $event.offsetY
	isDrawing.value = true
	ctx.value.beginPath()
	ctx.value.lineWidth = lineSize.value;
	ctx.value.strokeStyle = activeColor.value;
	ctx.value.fillStyle = activeColor.value;
	snapshot.value = ctx.value.getImageData(0, 0, canvasRef.value.width, canvasRef.value.height)
}

const onStopDraw = () => {
	isSelect.value = false
	isDrawing.value = false;
	// let data = JSON.stringify(ctx.value.getImageData(0, 0, canvasRef.value?.width, canvasRef.value?.height));
	addToCachCanvas()
}

const addToCachCanvas = () => {
	if (isDrawAction.value || activeTool.value === "text") {
		if (historyCach.value.length > 14) {
			historyCach.value.shift()
		}
		historyCach.value.push(ctx.value.getImageData(0, 0, canvasRef.value?.width, canvasRef.value?.height))
		currentCanvasIndex.value = historyCach.value.length - 1
		isDrawAction.value = false
	}
}

const moveCanvasElement = ($event) => {
	if (activeTool.value === "text" && isSelect.value) {
		$event.target.offsetX = prevMouseX.value
		$event.target.offsetY = prevMouseY.value
	}
}

const onPasteElement = ($event) => {
	console.log($event)
	// var img = new Image();
	// img.src = "http://upload.wikimedia.org/wikipedia/commons/d/d2/Svg_example_square.svg";
	// // img.onload = function() {
	// // 	ctx.value.drawImage(img, 0, 0);
	// // }
	// ctx.value.drawImage(img, 5, 5);
}


// window.addEventListener("paste", onPasteElement)
const onDraw = ($event) => {
	if (!isDrawing.value) {
		if (isSelect.value) {
			currentPositionY.value = $event.offsetY
			currentPositionX.value = $event.offsetX
		}
		return;
	} else {
		ctx.value.putImageData(snapshot.value, 0, 0)
		if (activeTool.value === "pencil" || activeTool.value === "clear") {
			ctx.value.strokeStyle = activeTool.value === "clear" ? "#fff" : activeColor.value
			ctx.value.lineTo($event.offsetX, $event.offsetY)
			ctx.value.stroke()
		} else if (activeTool.value === "rectangle") {
			onDrawRectangle($event)
		} else if (activeTool.value === "triangle") {
			onDrawTriangle($event)
		} else if (activeTool.value === "circle") {
			onDrawCircle($event)
		} else if (activeTool.value === "line") {
			onDrawLine($event)
		} else if (activeTool.value === "cursor") {
			moveCanvasElement($event)
		} else {
			return;
		}
		isDrawAction.value = true;
	}
}

const onClickCanvasElement = ($event) => {
	if (activeTool.value === "cursor") {
		// console.log($event)
	} else if (activeTool.value === "text") {
		currentAction.value = "writing"
		onSetText($event)
	}
}

const onDrawRectangle = ($event) => {
	if (fillFigure.value) {
		ctx.value.fillRect(
			$event.offsetX,
			$event.offsetY,
			prevMouseX.value - $event.offsetX,
			prevMouseY.value - $event.offsetY,
		)
	} else {
		ctx.value.strokeRect(
			$event.offsetX,
			$event.offsetY,
			prevMouseX.value - $event.offsetX,
			prevMouseY.value - $event.offsetY,
		)
	}
}
const onDrawTriangle = ($event) => {
	ctx.value.beginPath()
	ctx.value.moveTo(prevMouseX.value, prevMouseY.value)
	ctx.value.lineTo($event.offsetX, $event.offsetY)
	ctx.value.lineTo(prevMouseX.value * 2 - $event.offsetX, $event.offsetY)
	ctx.value.closePath()
	fillFigure.value ? ctx.value.fill() : ctx.value.stroke()
}
const onDrawCircle = ($event) => {
	ctx.value.beginPath()
	let radius = Math.sqrt(Math.pow((prevMouseX.value - $event.offsetX), 2) + Math.pow((prevMouseY.value - $event.offsetY), 2))
	ctx.value.arc(prevMouseX.value, prevMouseY.value, radius, 0, 2 * Math.PI)
	fillFigure.value ? ctx.value.fill() : ctx.value.stroke()
}
const onDrawLine = ($event) => {
	ctx.value.beginPath()
	ctx.value.moveTo(prevMouseX.value, prevMouseY.value)
	ctx.value.lineTo($event.offsetX, $event.offsetY)
	ctx.value.closePath()
	ctx.value.stroke()
}
const onSetText = ($event) => {
	let textarea = document.createElement("textarea")
	textarea.style.top = `${$event.offsetY}px`
	textarea.style.left = `${$event.offsetX + 200}px`
	textarea.style.position = "fixed"
	textarea.style.border = "1px solid black"
	$event.target.style.color = activeColor.value;
	$event.target.style.fontSize = `${fontSize.value}px`;
	if (!isFocusTextarea.value) {
		canvasWrapperRef.value.appendChild(textarea)
		textarea.focus()
		textarea.addEventListener("click", onClickTextArea)
		textarea.addEventListener("blur", onBlurTextArea)
		textarea.addEventListener("input", onInputTextArea)
		textarea.addEventListener("mousedown", () => isSelect.value = true)
		textarea.addEventListener("mouseup", () => isSelect.value = false)
		textarea.addEventListener("mousemove", moveCanvasElement)
		isFocusTextarea.value = true;
		listTextElements.value.push(textarea)
	}
}

const onUpdateElementData = ($event) => {
	return
}

const onBlurTextArea = ($event) => {
	$event.target.classList.add('blur-text-area')
	$event.target.style.color = activeColor.value;
	$event.target.style.fontSize = `${fontSize.value}px`;
	$event.target.classList.add('blur-text-area')
	if (!$event.target?.value?.length) {
		canvasWrapperRef.value.removeChild($event.target)
	}
};
const onClickTextArea = ($event) => {
	if (activeTool.value === "cursor" || activeTool.value === "text") {
		$event.target.classList.remove('blur-text-area')
		$event.target.focus()
		$event.target.style.color = activeColor.value;
		$event.target.style.fontSize = `${fontSize.value}px`;
	}
}
const onInputTextArea = ($event) => {
	$event.target.style.color = activeColor.value;
	$event.target.style.fontSize = `${fontSize.value}px`;
}

// const drawText = ($event) => {
// 	console.log($event)
// 	console.log($event.target.width)
// 	console.log($event.target.height)
// 	console.log($event.target.style.top)
// 	console.log($event.target.style.top)
// 	console.log($event.target.value)
// 	console.log($event.offsetX)
// 	console.log($event.offsetY)
// 	ctx.value.textBaseline = "top"
// 	ctx.value.textAlign = "left"
// 	ctx.value.font = fontSize.value;
// 	if ($event.target.value) {
// 		ctx.value.measureText($event.target.value).width = $event.target.innerWidth;
// 		ctx.value.measureText($event.target.value).height = $event.target.innerHeight;
// 		ctx.value.fillText($event.target.value, $event.target.style.left.split("px")[0], $event.target.style.top.split("px")[0]);
// 		document.body.removeChild($event.target)
// 		isFocusTextarea.value = false;
// 	}
// }

const saveElement = ($event) => {
	if (currentAction.value === "writing" && activeTool.value === "text") {
		// saveText($event)
		isFocusTextarea.value = false;
	}
}

// const saveText = ($event) => {
// 	ctx.value.textBaseline = "top"
// 	ctx.value.textAlign = "left"
// 	ctx.value.font = `normal ${fontSize.value}px serif`;
// 	ctx.value.fillText(textAreaRef.value?.value, currentPositionX.value, currentPositionY.value, );
// 	currentAction.value = "";
// }
const saveText = async ($event) => {
	currentAction.value = "";

	ctx.value.textBaseline = "top"
	ctx.value.textAlign = "left"
	// ctx.value.font = fontSize.value;
	// if ($event.target.value) {
	// 	ctx.value.measureText($event.target.value).width = $event.target.innerWidth;
	// 	ctx.value.measureText($event.target.value).height = $event.target.innerHeight;
	// 	ctx.value.fillText($event.target.value, $event.target.style.left.split("px")[0], $event.target.style.top.split("px")[0]);
	// 	document.body.removeChild($event.target)
	// 	isFocusTextarea.value = false;
	// }
	listTextElements.value.forEach((elem) => {
		console.log(elem.style)
		console.log(elem.style.fontSize)
		ctx.value.font = `normal ${elem.style.fontSize} serif`
		ctx.value.strokeStyle = activeColor.value;
		ctx.value.fillText(elem.value, elem.style.left.split("px")[0] - 200, elem.style.top.split("px")[0], )
		canvasWrapperRef.value.removeChild(elem)
	})
	listTextElements.value = []
}
const onBackCanvas = () => {
	if (historyCach.value.length) {
		if (currentCanvasIndex.value > 0) {
			currentCanvasIndex.value = currentCanvasIndex.value - 1
			ctx.value.putImageData(historyCach.value[currentCanvasIndex.value], 0, 0);
		} else {
			return
		}
	}
}
const onForwardCanvas = () => {
	if (historyCach.value.length) {
		if (currentCanvasIndex.value + 1 < historyCach.value.length) {
			currentCanvasIndex.value = currentCanvasIndex.value + 1
			ctx.value.putImageData(historyCach.value[currentCanvasIndex.value], 0, 0);
		} else {
			return
		}
	}
}

const clearCanvas = () => {
	ctx.value.clearRect(0, 0, canvasRef.value?.width, canvasRef.value?.height)
	setCanvasBackground()
	isDrawAction.value = true;
	addToCachCanvas()
}

const saveCanvas = async () => {
	await saveText();
	const link = document.createElement("a")
	link.download = `${Date.now()}.jpg`
	link.href = canvasRef.value?.toDataURL()
	link.click()
}
const setCanvasBackground = () => {
	ctx.value.fillStyle = "#fff"
	ctx.value.fillRect(0, 0, canvasRef.value?.width, canvasRef.value?.height)
	ctx.value.fillStyle = activeColor.value
}

const changeBoardTool = (value) => {
	currentAction.value = ""
	switch (value.type) {
		case "color":
			activeColor.value = value.value
			break;
		case "lineSize":
			lineSize.value = value.value
			break;
		case "fontSize":
			fontSize.value = value.value
			break;
		case "fillFigure":
			fillFigure.value = value.value
			break;
		case "tool":
			activeTool.value = value.value
			break;
	}
}

watch(() => activeTool.value, () => {
	if (activeTool.value === "text" || activeTool.value === "cursor") {
		listTextElements.value.forEach((elem) => {
			elem.disabled = false
		})
	} else {
		listTextElements.value.forEach((elem) => {
			elem.disabled = true
		})
	}
})

</script>

<style lang="scss">
.current-cursor-pencil {
	cursor: url("@/assets/images/pencil.svg"), auto!important;
}
.current-cursor-clear {
	cursor: url("@/assets/images/clear-icon.svg"), auto!important;
}
.blur-text-area {
	border: 1px solid rgba(128, 128, 128, 0.1) !important;
	outline: none!important;
	resize: none!important;
	&::-webkit-scrollbar {
		display: none;
	}
}
.board-main {
	height: 100%;
	width: 100%;
	display: flex;
	align-items: center;
	background-color: #e8e8e8;
	&__wrapper {
		width: 100%;
		padding: 0 1rem;
		height: 100%;
		margin: 0;
		display: flex;
		align-items: center;
		overflow: hidden!important;
		& textarea {
			height: auto;
			width: auto;
			background-color: inherit;
		}
		& #canvas {
			cursor: default;
			margin: 0;
			padding: 0;
			background-color: white;
			width: 100%;
			height: 99%;
		}
	}
}
</style>
