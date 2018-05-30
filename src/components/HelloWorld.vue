<template>
    <div class="container ">
        <div class="tool">
            <div class="nav-group">
                <div class="nav-group__title">aggregate functions</div>
                <ul class="pure-menu-list" style="height: auto;">
                    <li class="nav-item"><a draggable="true" @dragstart="drag">avg</a></li>
                    <li class="nav-item"><a draggable="true" @dragstart="drag">count</a></li>
                    <li class="nav-item"><a draggable="true" @dragstart="drag">max</a></li>
                    <li class="nav-item"><a draggable="true" @dragstart="drag">sum</a></li>
                </ul>
            </div>
            <div class="nav-group">
                <div class="nav-group__title">conversion functions</div>
                <ul class="pure-menu-list" style="height: auto;">
                    <li class="nav-item"><a draggable="true" @dragstart="drag">boolean</a></li>
                    <li class="nav-item"><a draggable="true" @dragstart="drag">number</a></li>
                    <li class="nav-item"><a draggable="true" @dragstart="drag">string</a></li>
                    <li class="nav-item"><a draggable="true" @dragstart="drag">parse-date</a></li>
                </ul>
            </div>
        </div>
        <div ref="paper" id="paper" @drop="drop" @dragover="allowDrop"></div>
    </div>
</template>

<script>
    import joint from 'jointjs/dist/joint'
    import response from '@/data.json'

    let container = {
        width: 320,
        height: 1000,
        padding: {
            x: 0,
            y: 25
        }
    }

    let nodeStyle = {
        width: 280,
        height: 40,
        tierWidth: 20
    }

    let margin = {
        x: 10,
        y: 30
    }
    let nodeMargin = {
        x: 20,
        y: 10
    }

    /**
     * 初始化画布
     */
    function createPaper () {
        let graph = new joint.dia.Graph;

        let paper = new joint.dia.Paper({
            el: document.getElementById('paper'),
            gridSize: 1,
            model: graph,
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
                return sourceMagnet != targetMagnet;
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

        return {
            graph,
            paper
        }
    }

    /**
     * 创建两边容器
     * @param paperWidth
     */
    function createContainer (paperWidth) {
        let x = margin.x, y = margin.y;
        if (paperWidth) {
            x = paperWidth - container.width - margin.x;
        }

        class containerModel extends joint.shapes.devs.Model {
            constructor (opt) {
                super(opt)
            }
        }

        return new containerModel({
            position: {
                x: x,
                y: y
            },
            size: {
                width: container.width,
                height: container.height
            },
            attrs: {
                'text': {
                    fill: '#24aebf'
                }
            }
        })
    }

    /**
     * 创建规则盒子
     * @param x
     * @param y
     */
    function createRuleBox (x, y) {
        class containerModel extends joint.shapes.devs.Model {
            constructor (opt) {
                super(opt)
                this.prop('type', 'rule');
                // this.attr('text/text', 'rules');
            }
        }

        return new containerModel({
            position: {
                x: x,
                y: y
            },
            size: {
                width: 150,
                height: 100
            },
            inPorts: ['1', '2'],
            outPorts: ['3']
        })
    }

    /**
     * 创建node节点
     * @param index
     * @param data
     * @param paperWidth
     */
    function createNode (index, data, paperWidth) {
        let x = margin.x + nodeMargin.x + data.tier * nodeStyle.tierWidth, y = margin.y + nodeMargin.y + container.padding.y;
        if (paperWidth) {
            x = paperWidth - container.width - margin.x + nodeMargin.x + data.tier * nodeStyle.tierWidth;
        }
        if (index) {
            y += (nodeStyle.height + nodeMargin.y) * index
        }

        class nodeModel extends joint.shapes.devs.Model {
            constructor (opt) {
                super(opt)
                this.prop('index', index);
                this.prop('tier', data.tier);
                this.attr('text/text', data.value);
                this.prop('id', data.value);
                this.prop('data', data);
                if (data.children.length) {
                    this.markup = '<g class="rotatable"><rect class="body"/><polygon /><text class="label"/></g>'
                }
                // else {
                //     this.markup = '<g class="rotatable"><rect class="body"/><rect class="parcel" /><text class="label"/></g>'
                // }
            }

            setExpand (expand) {
                return this.prop('expand', expand)
            }

            expandFn (boole) {
                if (boole) {
                    this.attr('polygon/points', '0,10 10,10 10, 0')
                } else {
                    this.attr('polygon/points', '0,10 10,5 0, 0')
                }
            }

            setType (type) {
                return this.prop('type', type)
            }

            get data () {
                return this.prop('data')
            }

            get type () {
                return this.prop('type')
            }

            get tier () {
                return this.prop('tier')
            }

            get expand () {
                return this.prop('expand')
            }
        }

        return new nodeModel({
            position: {
                x: x,
                y: y
            },
            size: {
                width: nodeStyle.width - data.tier * nodeStyle.tierWidth,
                height: nodeStyle.height
            },
            attrs: {
                // '.expand': {
                //     width: 6,
                //     height: 1,
                //     fill: '#000000',
                //     x: 18,
                //     y: 19
                // },
                '.body': {
                    rx: 5,
                    ry: 5
                },
                '.parcel': {
                    width: 10,
                    height: 10,
                    fill: '#000000',
                    refX: .05,
                    refY: .3
                },
                'polygon': {
                    points: '0,10 10,5 0, 0',
                    fill: '#000000',
                    refX: .05,
                    refY: .3
                }
            }
        })
    }

    /**
     * 连接
     * @param source
     * @param target
     * @param graph
     */
    function connectFn (source, target, graph) {
        class link extends joint.shapes.devs.Link {
            constructor (opt) {
                super(opt);
            }
        }

        new link({
            source: {
                id: source,
                port: ''
            },
            target: {
                id: target,
                port: ''
            },
            attrs: {
                '.connection': {
                    'stroke-width': 1,
                    targetMarker: {
                        type: 'path',
                        fill: '#7c68fc',
                        stroke: 'none',
                        d: 'M 20 -10 0 0 20 10 z'
                    }
                }
                // '.marker-source': { fill: '#4b4a67', stroke: '#4b4a67', d: 'M 10 0 L 0 5 L 10 10 z' },
                // '.marker-target': { fill: '#4b4a67', stroke: '#4b4a67', d: 'M 10 0 L 0 5 L 10 10 z' }
            }
        }).addTo(graph).reparent();
    };

    const connects = [
        {
            'sourceId': 'target-data1-1',
            'targetId': 'source-data1'
        },
        {
            'sourceId': 'source-data2-1',
            'targetId': 'target-data1-1-1'
        },
        {
            'sourceId': 'source-data1-1',
            'targetId': 'target-data2-1-1'
        },
        {
            'sourceId': 'source-data2',
            'targetId': 'target-data2-2'
        },
        {
            'sourceId': 'target-data2',
            'targetId': 'source-data2-1'
        },
        {
            'sourceId': 'target-data1',
            'targetId': 'source-data3'
        },
        {
            'sourceId': 'source-data2-2',
            'targetId': 'target-data3'
        }
    ]

    export default {
        name: 'HelloWorld',
        data () {
            return {
                graph: null,
                paper: null,
                paperWidth: 0
            }
        },
        mounted () {
            const paperObj = createPaper();
            this.graph = paperObj.graph;
            this.paper = paperObj.paper;
            this.paperWidth = this.getWidth();
            this.init();
        },
        methods: {
            getWidth () {
                return this.$refs['paper'].offsetWidth
            },
            init () {
                const leftContainer = createContainer();
                const rightContainer = createContainer(this.paperWidth)
                leftContainer.attr('text/text', 'FIRE数据源')
                rightContainer.attr('text/text', 'HIS数据源')
                const nodes = [leftContainer, rightContainer];
                let sourceBum = 0;
                let targetBum = 0;

                const setSourceChart = (data) => {
                    lodash.forEach(data, (item, index) => {
                        const nodeItem = createNode(sourceBum, item);
                        nodeItem
                            .set('outPorts', [''])
                            .setType('source')
                            .setExpand(false);
                        // item.children.length ? nodeItem.setExpand(true) : nodeItem.setExpand(false)
                        // leftContainer.embed(nodeItem);
                        nodes.push(nodeItem);
                        sourceBum++;
                        // if (!item.children.length)
                        //     return;
                        // return setSourceChart(item.children)
                    })
                }

                const setTargetChart = (data) => {
                    lodash.forEach(data, (item, index) => {
                        const nodeItem = createNode(targetBum, item, this.paperWidth);
                        nodeItem
                            .set('inPorts', [''])
                            .setType('target')
                            .setExpand(false);
                        // item.children.length ? nodeItem.setExpand(true) : nodeItem.setExpand(false)
                        // rightContainer.embed(nodeItem);
                        nodes.push(nodeItem);
                        targetBum++;
                        // if (!item.children.length)
                        //     return;
                        // return setTargetChart(item.children)
                    })
                }

                setTimeout(() => {
                    setSourceChart(response.source)
                    setTargetChart(response.target)

                    this.graph.addCells(nodes)
                    // 连接
                    const sourceList = lodash.map(response.source, item => item.value)
                    const targetList = lodash.map(response.target, item => item.value)
                    lodash.forEach(connects, connect => {
                        if (sourceList.indexOf(connect.sourceId) !== -1 && targetList.indexOf(connect.targetId) !== -1) {
                            connectFn(connect.sourceId, connect.targetId, this.graph)
                        }
                        if (sourceList.indexOf(connect.targetId) !== -1 && targetList.indexOf(connect.sourceId) !== -1) {
                            connectFn(connect.sourceId, connect.targetId, this.graph)
                        }
                    });
                    // this.graph.addCell(createRuleBox())
                }, 100)

                this.paper.on('cell:pointerclick', this.treeClick)
                this.paper.on('link:connect', this.connectFn)
            },
            /**
             * 收缩tree
             * @param type
             * @param cellView
             * @param models
             */
            operationCloseTree (type, cellView, models) {
                models = lodash.filter(models, m => m.type === type);
                let startIndex = lodash.findIndex(models, m => m.cid === cellView.cid);
                let endIndex = 0;
                let removeNodes = [];
                for (let i = startIndex + 1; i < models.length; i++) {
                    if (models[i].tier <= cellView.tier) {
                        endIndex = i;
                        break;
                    } else {
                        removeNodes.push(models[i])
                    }
                }

                let moveNodes = models.slice(endIndex);

                lodash.forEach(moveNodes, node => {
                    startIndex++;
                    let x = 0;
                    if (type === 'source') {
                        x = margin.x + nodeMargin.x + node.tier * nodeStyle.tierWidth;
                    } else {
                        x = this.paperWidth - container.width - margin.x + nodeMargin.x;
                    }
                    let y = margin.y + nodeMargin.y + container.padding.y;
                    y += (nodeStyle.height + nodeMargin.y) * startIndex
                    node.position(x, y)
                })
                this.graph.removeCells(removeNodes);
            },
            /**
             * 展开tree
             * @param type
             * @param cellView
             * @param models
             */
            operationExpandTree (type, cellView, models) {
                models = lodash.filter(models, m => m.type === type);
                let otherModels;
                if (type === 'source') {
                    otherModels = lodash.filter(this.graph.getCells(), m => m.type === 'target');
                } else {
                    otherModels = lodash.filter(this.graph.getCells(), m => m.type === 'source');
                }

                let startIndex = lodash.findIndex(models, m => m.cid === cellView.cid);
                let addNodes = [];
                let currentZ = cellView.get('z');
                let cuntZ = currentZ + cellView.data.children.length;
                const beanModels = this.graph.getCells();

                lodash.forEach(beanModels, node => {
                    const z = node.get('z');
                    if (z > currentZ) {
                        cuntZ++;
                        node.set('z', cuntZ);
                    }
                })

                lodash.forEach(cellView.data.children, node => {
                    startIndex++;
                    let nodeItem;
                    if (type === 'source') {
                        nodeItem = createNode(startIndex, node);
                        nodeItem
                            .set('outPorts', [''])
                            .setType('source');
                    } else {
                        nodeItem = createNode(startIndex, node, this.paperWidth);
                        nodeItem
                            .set('inPorts', [''])
                            .setType('target');
                    }
                    currentZ++;
                    nodeItem.set('z', currentZ)
                    addNodes.push(nodeItem)
                    models.splice(startIndex, 0, nodeItem)
                })

                for (let i = startIndex + 1; i < models.length; i++) {
                    let x = 0;
                    if (type === 'source') {
                        x = margin.x + nodeMargin.x + models[i].tier * nodeStyle.tierWidth;
                    } else {
                        x = this.paperWidth - container.width - margin.x + nodeMargin.x + models[i].tier * nodeStyle.tierWidth;
                    }
                    let y = margin.y + nodeMargin.y + container.padding.y;
                    y += (nodeStyle.height + nodeMargin.y) * i
                    models[i].position(x, y)
                }

                this.graph.addCells(addNodes);

                // connect
                let sourceList = lodash.map(cellView.data.children, node => node.value)
                let otherList = lodash.map(otherModels, node => node.id);

                lodash.forEach(connects, connect => {
                    if (sourceList.indexOf(connect.sourceId) !== -1 && otherList.indexOf(connect.targetId) !== -1) {
                        connectFn(connect.sourceId, connect.targetId, this.graph)
                    }
                    if (sourceList.indexOf(connect.targetId) !== -1 && otherList.indexOf(connect.sourceId) !== -1) {
                        connectFn(connect.targetId, connect.sourceId, this.graph)
                    }
                })
            },
            /**
             * 画布点击列表时触发
             * @param cellView
             */
            treeClick (cellView) {
                const expand = cellView.model.expand;
                const type = cellView.model.type;

                if (!cellView.model.data || !cellView.model.data.children.length)
                    return;

                if (expand) {
                    this.operationCloseTree(type, cellView.model, cellView.model.collection.models);
                    cellView.model.setExpand(false);
                    cellView.model.expandFn(false);
                } else {
                    this.operationExpandTree(type, cellView.model, cellView.model.collection.models);
                    cellView.model.setExpand(true);
                    cellView.model.expandFn(true);
                }
            },
            /**
             * 连接时触发
             * @param linkView
             */
            connectFn (linkView) {
                //do something
                linkView.model.attr({
                    '.connection': {
                        'stroke-width': 1,
                        targetMarker: {
                            type: 'path',
                            fill: '#7c68fc',
                            stroke: 'none',
                            d: 'M 20 -10 0 0 20 10 z'
                        }
                    }
                })
            },
            drop (evt) {
                const text = evt.dataTransfer.getData("value")
                const node = createRuleBox(evt.layerX, evt.layerY)
                node.attr('text/text', text)
                this.graph.addCell(node)
            },
            drag (evt) {
                evt.dataTransfer.setData("value", evt.target.innerHTML);
            },
            allowDrop (ev) {
                ev.preventDefault();
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
    .container {
        height: 100%;
        display: flex;
    }

    .container .tool {
        flex: 1;
        height: 100%;
        overflow-y: auto;
        overflow-x: hidden;
        padding: 10px;
        border-right: 1px solid #ccc;
    }

    #paper {
        flex: 9;
        height: 100%;
        width: 100%
    }

    .joint-paper.joint-theme-default {
        width: 100% !important;
        height: 100% !important;
    }

    .joint-element {
        cursor: default;
    }

    .port-body.joint-port-body {
        stroke: #ffffff;
        stroke-width: 3px;
        fill: #7c68fc;
    }

    .nav-group__title {
        font-size: 12px;
        color: #999;
        line-height: 26px;
        margin-top: 15px;
    }

    .pure-menu-list {
        padding: 0;
        margin: 0;
        overflow: hidden;
    }

    .pure-menu-list li {
        list-style: none;
    }

    .nav-item a {
        display: block;
        height: 40px;
        color: #444;
        line-height: 40px;
        font-size: 14px;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
        font-weight: 400;
        margin: 0;
        padding: 0;
        text-decoration: none;
        position: relative;
        transition: .15s ease-out;
        cursor: default;
        user-select: none;
    }
</style>
