<template>
  <div class="h-screen w-screen flex items-center justify-center">
    <div ref="pixi" class="w-50vw h-50vw" />
  </div>
</template>

<script setup>
/* eslint-disable import/namespace */

import * as PIXI from 'pixi.js'
import { onMounted } from 'vue'
const pixi = ref()
let app = null
let width = 0
let height = 0
let gridWidth = 0
let gridHeight = 0
let squareSize = 0
const amountOfSquaresPerWidth = 20
let selectedGraphics = null
const line = new PIXI.Graphics()
const shape = []

const zoom = (event) => {
  if (event.wheelDelta > 0) {
    app.stage.scale.x = app.stage.scale.x * 1.1
    app.stage.scale.y = app.stage.scale.y * 1.1
  }
  else {
    app.stage.scale.x = app.stage.scale.x * 0.9
    app.stage.scale.y = app.stage.scale.y * 0.9
  }
}
const setupPanning = () => {
  const bg = new PIXI.Sprite(PIXI.Texture.WHITE)
  // Set it to fill the screen
  bg.width = app.screen.width
  bg.height = app.screen.height
  // Tint it to whatever color you want, here red
  bg.tint = 0xFF0000
  bg.alpha = 0
  bg.zIndex = 0
  // Add a click handler
  bg.interactive = true
  // Add it to the stage as the first object
  app.stage.addChild(bg)
  app.stage.interactive = true

  let isDragging = false
  let prevX; let prevY

  bg.mousedown = function(moveData) {
    const pos = moveData.data.global
    prevX = pos.x; prevY = pos.y
    isDragging = true
  }

  bg.mousemove = function(moveData) {
    if (!isDragging)
      return
    const pos = moveData.data.global
    const dx = pos.x - prevX
    const dy = pos.y - prevY

    app.stage.position.x += dx
    app.stage.position.y += dy
    prevX = pos.x; prevY = pos.y
  }

  bg.mouseup = function(moveDate) {
    isDragging = false
  }
}

const selectGraphics = (graphics) => {
  if (selectedGraphics) {
    line.moveTo(selectedGraphics.x, selectedGraphics.y)
    line.lineStyle(2, 0xFFFFFF, 1)
    line.lineTo(graphics.x, graphics.y)
    app.stage.addChild(line)
    selectedGraphics.scale.set(1)
    // console.log(`from - ${selectedGraphics.x} - ${selectedGraphics.y}`)
    // console.log(`to - ${graphics.x} - ${graphics.y}`)
    // console.log(line)
  }
  shape.push({ x: Math.round((graphics.x - 50) / squareSize), y: Math.round((graphics.y - 50) / squareSize) })

  selectedGraphics = graphics
  graphics.scale.set(2)
}

const init = () => {
  width = pixi.value.offsetWidth
  height = pixi.value.offsetHeight
  gridWidth = width - 100
  gridHeight = height - 100
  app = new PIXI.Application(
    {
      autoResize: true,
      resolution: devicePixelRatio,
    },
  )
  app.renderer.resize(width, height)
  setupPanning()

  squareSize = gridWidth / amountOfSquaresPerWidth
  const amounthPerHeight = Math.floor(gridHeight / (gridWidth / amountOfSquaresPerWidth))
  for (let j = 0; j < amounthPerHeight; j++) {
    for (let i = 0; i < amountOfSquaresPerWidth; i++) {
      const square = new PIXI.Graphics()
      square.lineStyle(2, 0xFFFF00, 1)
      square.drawRect(squareSize * i + 50, squareSize * j + 50, squareSize, squareSize)
      app.stage.addChild(square)
    }
  }

  for (let j = 0; j < amounthPerHeight + 1; j++) {
    for (let i = 0; i < amountOfSquaresPerWidth + 1; i++) {
      const graphics = new PIXI.Graphics()
      graphics.beginFill(0xDE3249, 1)
      graphics.drawCircle(squareSize * i + 50, squareSize * j + 50, amountOfSquaresPerWidth / 5)
      graphics.endFill()
      graphics.pivot.set(squareSize * i + 50, squareSize * j + 50)
      graphics.position.set(squareSize * i + 50, squareSize * j + 50)
      graphics.zIndex = 1
      graphics.interactive = true
      graphics
        .on('pointerover', () => {
          graphics.scale.set(2)
        })
        .on('pointerout', () => {
          if (selectedGraphics !== graphics)
            graphics.scale.set(1)
        })
        .on('pointerdown', () => { selectGraphics(graphics) })

      app.stage.addChild(graphics)
    }
  }

  pixi.value.appendChild(app.view)
}

onMounted(() => {
  init()
  pixi.value.addEventListener('wheel', zoom)
})

</script>
