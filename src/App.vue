<template>
    <div id="app">
        <div id="header">
            <div class="spacer"></div>
            <!-- <button class="action">
                <span class="material-icons-round">file_download</span>
            </button> -->
            <button class="action">
                <span class="material-icons-round">code</span>
            </button>
        </div>

        <div id="tools">
            <button class="tool">
                <span class="material-icons-round">near_me</span>
            </button>
            <button class="tool">
                <span class="material-icons-round">edit</span>
            </button>

            <div class="spacer"></div>

            <button class="tool" :class="{'active': window.layer === 'points'}" @click="window.layer = 'points'">
                <span class="material-icons-round">layers</span>
            </button>
            <button class="tool" :class="{'active': window.layer === 'debug'}" @click="window.layer = 'debug'">
                <span class="material-icons-round">info</span>
            </button>
        </div>

        <div id="layers">
            <div v-show="window.layer === 'points'">
                <div class="point" v-for="(point, i) in points" :key="'point_layer_'+i">
                    <span class="material-icons-round">linear_scale</span>
                    <div class="type">{{point.type}}</div>
                </div>
            </div>

            <div v-show="window.layer === 'debug'">
                <div class="padding">
                    Bounds Top: <b>{{canvas.bounds.top}}</b><br>
                    Bounds Left: <b>{{canvas.bounds.left}}</b><br>
                </div>
                <hr>
                <div class="padding">
                    Cursor X: <b>{{cursor.x}}</b><br>
                    Cursor Y: <b>{{cursor.y}}</b><br>
                    Canvas X: <b>{{canvas.cursorX}}</b><br>
                    Canvas Y: <b>{{canvas.cursorY}}</b><br>
                    Focused Point: <b>{{typeof focusedPoint !== 'object' ? focusedPoint : 'none'}}</b><br>
                </div>
                <hr>
                <div class="padding">
                    Is Grabbing: <b>{{grab.isDown}}</b><br>
                    Start X: <b>{{grab.startX}}</b><br>
                    Start Y: <b>{{grab.startY}}</b><br>
                    Current X: <b>{{grab.currentX}}</b><br>
                    Current Y: <b>{{grab.currentY}}</b><br>
                    ΔX: <b>{{grab.deltaX}}</b><br>
                    ΔY: <b>{{grab.deltaY}}</b><br><br>
                    Value Start X: <b>{{grab.value.startX}}</b><br>
                    Value Start Y: <b>{{grab.value.startY}}</b><br>
                </div>
            </div>
        </div>

        <div id="canvas-wrapper" ref="canvasWrapper" draggable="false" @mousedown="newAt($event)">
            <svg draggable="false" class="canvas" ref="canvas">
                <path draggable="false" :d="pathString" fill="#1e90ff10" stroke="#1e90ff" stroke-width="1" />
            </svg>
            <div draggable="false" class="point" :class="{'focused': focusedPoint === i}" v-for="(point, i) in drawablePoints" :key="i" :style="`left: ${point.x}px; top: ${point.y}px;`" @mousedown.stop="downOn($event, i)"></div>
        </div>
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

                window: {
                    layer: 'points',
                },

                canvas: {
                    bounds: {},
                    cursorX: 0,
                    cursorY: 0,
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
                if (this.focusedPoint === this.points.length - 1 && this.points[this.points.length - 1].type !== 'close')
                {
                    this.points.push({type: 'close', isAbsolute: true, x: 0, y: 0})
                }
                else
                {
                    this.grab.isDown = true
                    this.grab.startX = event.clientX
                    this.grab.startY = event.clientY
                    this.grab.deltaX = 0
                    this.grab.deltaY = 0
    
                    this.grab.value.startX = this.points[point].x
                    this.grab.value.startY = this.points[point].y
                }

                this.focusedPoint = point
            },

            newAt(event) {
                if (this.focusedPoint === this.points.length - 1)
                {
                    this.points.push({type: 'line', isAbsolute: true, x: this.canvas.cursorX, y: this.canvas.cursorY})
                    this.focusedPoint = this.points.length - 1
                }
            },

            move(event) {
                this.cursor.x = event.clientX
                this.cursor.y = event.clientY

                this.canvas.cursorX = this.cursor.x - this.canvas.bounds.left || 0
                this.canvas.cursorY = this.cursor.y - this.canvas.bounds.top || 0

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
        height: 100%
        --primary: #1e90ff
        --secondary: #3742fa

    *
        box-sizing: border-box

        ::-webkit-scrollbar
            width: 10px

        ::-webkit-scrollbar-track
            background: white

        ::-webkit-scrollbar-thumb
            background: #ccc
            border: 3px solid white
            border-radius: 6px

    fieldset
        border: 1px solid #ddd
        margin: 5px
        border-radius: 8px
        padding: 10px
</style>

<style lang="sass" scoped>
    #app
        width: 100%
        height: 100%
        display: grid
        grid-template-rows: 50px 1fr
        grid-template-columns: 50px 260px 1fr
        grid-template-areas: "header header header" "tools layers canvas"
        background: #f4f4f4
        gap: 3px 3px

        #header
            grid-area: header
            background: white
            display: flex
            gap: 3px

            .spacer
                height: 100%
                flex: 1

            .action
                height: 50px
                width: 50px
                display: grid
                place-content: center
                border: none
                background: none
                cursor: pointer

                &:hover
                    color: var(--secondary)

        #tools
            grid-area: tools
            background: white
            display: flex
            flex-direction: column
            
            .spacer
                flex: 1
                width: 100%
            
            .tool
                height: 50px
                width: 50px
                display: grid
                place-content: center
                border: none
                background: none
                cursor: pointer

                span
                    font-size: 22px !important

                &:hover,
                &.active
                    color: var(--secondary)

        #layers
            grid-area: layers
            background: white
            overflow: hidden
            overflow-y: scroll

            .padding
                padding: 10px

            hr
                border: none
                border-bottom: 3px solid #f4f4f4
                margin: 0

            .point
                height: 40px
                display: flex
                align-items: center

                span
                    width: 50px
                    font-size: 20px
                    text-align: center
                    opacity: 0.5

        #canvas-wrapper
            grid-area: canvas
            background: white
            position: relative
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
                border: 1px solid var(--primary)
                transform: translate(-50%, -50%)
                user-select: none

                &.focused
                    background: var(--primary)
</style>