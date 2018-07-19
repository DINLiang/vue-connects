<template>
    <div style="height: 100%; width: 100%">
        <button @click="save">保存</button>
        <div id="paper" style="height: 100%; width: 100%">
        </div>
    </div>
</template>

<script>
    import joint from 'jointjs/dist/joint';

    export default {
        name: "test",
        data() {
            return {
                beanGraph: null,
                paper: null,
                cells: []
            }
        },
        mounted() {
            this.beanGraph = new joint.dia.Graph;
            this.paper = new joint.dia.Paper({
                el: document.getElementById('paper'),
                gridSize: 1,
                model: this.beanGraph,
                snapLinks: true,
                linkPinning: false,
                embeddingMode: true,
                highlighting: {
                    'default': {
                        name: 'stroke',
                        options: {
                            padding: 6
                        }
                    },
                    'embedding': {
                        name: 'addClass',
                        options: {
                            className: 'highlighted-parent'
                        }
                    }
                },

                defaultLink: new joint.dia.Link({
                    attrs: {
                        '.connection': {
                            'stroke-width': 2,
                            targetMarker: {
                                type: 'path',
                                fill: '#7c68fc',
                                stroke: 'none',
                                d: 'M 20 -10 0 0 20 10 z'
                            }
                        }
                    },
                    toolMarkup: [
                        '<g class="link-tool">',
                        '<g class="tool-remove" event="tool:remove">',
                        '<circle r="11" />',
                        '<path transform="scale(.8) translate(-16, -16)" d="M24.778,21.419 19.276,15.917 24.777,10.415 21.949,7.585 16.447,13.087 10.945,7.585 8.117,10.415 13.618,15.917 8.116,21.419 10.946,24.248 16.447,18.746 21.948,24.248z"/>',
                        '<title>删除链接</title>',
                        '</g>',
                        '</g>'
                    ].join('')
                }),

                validateEmbedding (childView, parentView) {
                    return parentView.model instanceof joint.shapes.devs.Model;
                },

                validateConnection (sourceView, sourceMagnet, targetView, targetMagnet) {
                    return sourceMagnet !== targetMagnet;
                },

                // 元素交互启用
                // arrowheadMove elementMove addLinkFromMagnet
                interactive (element) {
                    let flag = false;
                    if (element.model.get('type') === 'rule') {
                        flag = true
                    }
                    return {
                        elementMove: flag,
                        vertexAdd: false
                    }
                }
            });
        }
    }
</script>

<style scoped>

</style>