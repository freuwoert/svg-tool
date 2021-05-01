<template>
    <div id="app">
        <div class="canvas-wrapper" ref="canvasWrapper" draggable="false">
            <svg draggable="false" class="canvas" ref="canvas">
                <path draggable="false" :d="pathString" fill="#1e90ff10" stroke="#1e90ff" stroke-width="1" />
            </svg>
            <div draggable="false" class="point" :class="{'focused': focusedPoint === i}" v-for="(point, i) in drawablePoints" :key="i" :style="`left: ${point.x}px; top: ${point.y}px;`" @mousedown="downOn($event, i)"></div>
        </div>

        <fieldset>
            Bounds Top: <b>{{canvas.bounds.top}}</b><br>
            Bounds Left: <b>{{canvas.bounds.left}}</b><br>
        </fieldset>
        <fieldset>
            Cursor X: <b>{{cursor.x}}</b><br>
            Cursor Y: <b>{{cursor.y}}</b><br>
            Canvas X: <b>{{cursor.x - canvas.bounds.left}}</b><br>
            Canvas Y: <b>{{cursor.y - canvas.bounds.top}}</b><br>
            Focused Point: <b>{{typeof focusedPoint !== 'object' ? focusedPoint : 'none'}}</b><br>
        </fieldset><br>
        <fieldset>
            Is Grabbing: <b>{{grab.isDown}}</b><br>
            Start X: <b>{{grab.startX}}</b><br>
            Start Y: <b>{{grab.startY}}</b><br>
            Current X: <b>{{grab.currentX}}</b><br>
            Current Y: <b>{{grab.currentY}}</b><br>
            ΔX: <b>{{grab.deltaX}}</b><br>
            ΔY: <b>{{grab.deltaY}}</b><br><br>
            Value Start X: <b>{{grab.value.startX}}</b><br>
            Value Start Y: <b>{{grab.value.startY}}</b><br>
        </fieldset>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                points: [
                    {type: 'move', isAbsolute: true, x: 100, y: 100},
                    {type: 'line', isAbsolute: true, x: 300, y: 100},
                    {type: 'line', isAbsolute: true, x: 200, y: 300},
                    // {type: 'close', isAbsolute: false, x: 0, y: 0},
                ],

                canvas: {
                    bounds: {},
                },

                cursor: {
                    x: 0,
                    y: 0,
                },

                grab: {
                    isDown: false,
                    startX: 0,
                    startY: 0,
                    currentX: 0,
                    currentY: 0,
                    deltaX: 0,
                    deltaY: 0,

                    value: {
                        startX: 0,
                        startY: 0,
                    },
                },

                focusedPoint: null,
            }
        },

        mounted() {
            let wrapper = this.$refs.canvasWrapper
            let canvas = this.$refs.canvas

            canvas.style.width = wrapper.offsetWidth + "px"
            canvas.style.height = wrapper.offsetHeight + "px"

            this.canvas.bounds = wrapper.getBoundingClientRect()

            window.addEventListener('mousemove', this.move)
            window.addEventListener('mouseup', this.up)
        },

        beforeDestroy() {
            window.removeEventListener('mousemove', this.move)
            window.removeEventListener('mouseup', this.up)
        },

        computed: {
            pathString() {
                let path = ''

                for (const point of this.points)
                {
                    switch (point.type)
                    {
                        case 'move':
                            path += `${point.isAbsolute ? 'M' : 'm'} ${point.x} ${point.y} `
                            break

                        case 'line':
                            path += `${point.isAbsolute ? 'L' : 'l'} ${point.x} ${point.y} `
                            break

                        case 'close':
                            path += `${point.isAbsolute ? 'Z' : 'z'} `
                            break
                    }
                }

                return path
            },

            drawablePoints() {
                return this.points.filter(e => ['move', 'line'].includes(e.type))
            },
        },

        methods: {
            downOn(event, point) {
                this.focusedPoint = point
                this.grab.isDown = true
                this.grab.startX = event.clientX
                this.grab.startY = event.clientY
                this.grab.deltaX = 0
                this.grab.deltaY = 0

                this.grab.value.startX = this.points[point].x
                this.grab.value.startY = this.points[point].y
            },

            move(event) {
                this.cursor.x = event.clientX
                this.cursor.y = event.clientY

                if (this.grab.isDown)
                {
                    this.grab.currentX = event.clientX
                    this.grab.currentY = event.clientY
                    this.grab.deltaX = this.grab.currentX - this.grab.startX
                    this.grab.deltaY = this.grab.currentY - this.grab.startY

                    if (typeof this.focusedPoint === 'number')
                    {
                        this.points[this.focusedPoint].x = this.grab.value.startX + this.grab.deltaX
                        this.points[this.focusedPoint].y = this.grab.value.startY + this.grab.deltaY
                    }
                }
            },

            up() {
                this.grab.isDown = false
            }
        },

        components: {}
    }
</script>

<style lang="sass">
    html, body
        margin: 0
        padding: 0
        font-family: 'Roboto'
</style>

<style lang="sass" scoped>
    .canvas-wrapper
        height: 400px
        width: calc(100% - 100px)
        background: #f4f4f4
        position: relative
        margin: 50px auto
        user-select: none

        .canvas
            position: absolute
            top: 50%
            left: 50%
            transform: translate(-50%, -50%)
            pointer-events: none
            user-select: none

        .point
            position: absolute
            height: 10px
            width: 10px
            border-radius: 100%
            background: white
            border: 1px solid #1e90ff
            transform: translate(-50%, -50%)
            user-select: none

            &.focused
                background: #1e90ff
</style>