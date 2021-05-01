<template>
    <div id="app">
        <div class="canvas-wrapper" ref="canvasWrapper">
            <svg class="canvas" ref="canvas">
                <path :d="pathString" fill="#1e90ff10" stroke="#1e90ff" stroke-width="1" />
            </svg>
            <div class="point" v-for="(point, i) in points" :key="i" :style="`left: ${point.x}px; top: ${point.y}px;`"></div>
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
                    {type: 'close', isAbsolute: false, x: 0, y: 0},
                ],
            }
        },

        mounted() {
            let wrapper = this.$refs.canvasWrapper
            let canvas = this.$refs.canvas

            canvas.style.width = wrapper.offsetWidth + "px"
            canvas.style.height = wrapper.offsetHeight + "px"
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

        .canvas
            position: absolute
            top: 50%
            left: 50%
            transform: translate(-50%, -50%)
            pointer-events: none

        .point
            position: absolute
            height: 10px
            width: 10px
            border-radius: 100%
            background: white
            border: 1px solid #1e90ff
            transform: translate(-50%, -50%)
</style>