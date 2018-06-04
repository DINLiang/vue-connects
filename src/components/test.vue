<template>
    <div id="paper" style="height: 100%; width: 100%">
    </div>
</template>

<script>
    import joint from 'jointjs/dist/joint';

    export default {
        name: "test",
        mounted() {
            let beanGraph = new joint.dia.Graph;
            let paper = new joint.dia.Paper({
                el: document.getElementById('paper'),
                gridSize: 1,
                model: beanGraph,
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
            beanGraph.addCells(window.cells)
        }
    }
</script>

<style scoped>

</style>