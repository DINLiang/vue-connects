<template>
    <div class="container ">
        <div class="tool">
            <button @click="save">保存</button>
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
    import joint from 'jointjs/dist/joint';
    import response from '@/data.json';

    const ruleMap = {
        avg: {
            inPorts: ['parent-context', 'values'],
            outPorts: ['result']
        },
        count: {
            inPorts: ['parent-context', 'nodes/rows', 'value'],
            outPorts: ['result']
        },
        max: {
            inPorts: ['parent-context', 'values'],
            outPorts: ['result']
        },
        sum: {
            inPorts: ['parent-context', 'values'],
            outPorts: ['result']
        },
        boolean: {
            inPorts: ['arg'],
            outPorts: ['result']
        },
        number: {
            inPorts: ['arg'],
            outPorts: ['result']
        },
        string: {
            inPorts: ['arg'],
            outPorts: ['result']
        },
        'parse-date': {
            inPorts: ['arg'],
            outPorts: ['parent-context', 'values']
        }
    };

    // const connects = [
        //     {
        //         'sourceId': 'target-data1-1',
        //         'targetId': 'source-data1'
        //     },
        //     {
        //         'sourceId': 'source-data2-1',
        //         'targetId': 'target-data1-1-1'
        //     },
        //     {
        //         'sourceId': 'source-data1-1',
        //         'targetId': 'target-data2-1-1'
        //     },
        //     {
        //         'sourceId': 'source-data2',
        //         'targetId': 'target-data2-2'
        //     },
        //     {
        //         'sourceId': 'target-data2',
        //         'targetId': 'source-data2-1'
        //     },
        //     {
        //         'sourceId': 'target-data1',
        //         'targetId': 'source-data3'
        //     },
        //     {
        //         'sourceId': 'source-data2-2',
        //         'targetId': 'target-data3'
        //     }
    // ];

    // 容器默认参数
    const container = {
        width: 320,
        height: 1000,
        padding: {
            x: 0,
            y: 25
        },
        margin: {
            x: 10,
            y: 30
        }
    };

    // tree节点初始化参数
    const nodeStyle = {
        width: 280,
        height: 40,
        tierWidth: 20,
        margin: {
            x: 20,
            y: 10
        }
    };

    // 规则默认参数
    const ruleNodeStyle = {
        width: 250,
        height: 30,
        titleHeight: 30,
        itemHeight: 30
    };

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
        let x = nodeStyle.margin.x, y = nodeStyle.margin.y;
        if (paperWidth) {
            x = paperWidth - container.width - nodeStyle.margin.x;
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
     * @param position
     */
    function createRuleBox (position) {
        class ruleBoxModel extends joint.shapes.basic.Generic {
            constructor (opt) {
                super(opt);
                this.prop('type', 'rule');
            }
        }

        return new ruleBoxModel({
            position: {
                x: position.x,
                y: position.y
            },
            size: {
                width: ruleNodeStyle.width,
                height: ruleNodeStyle.titleHeight
            },
            // markup: `<rect class="body"/>
            //         <text class="label"/>
            //         <text class="close" />
            //         <g class="options">
            //             <g class="left-container" />
            //             <g class="right-container" />
            //         </g>`,
            markupLeftOption:
                `<g>
                                <rect class="left-rect"/>
                                <text class="option-in-text"/>
                            </g>`,
            markupRightOption:
                `<g>
                                <rect class="right-rect"/>
                                <text class="option-out-text"/>
                            </g>`,
            markup: [{
                tagName: 'rect',
                className: 'body'
            }, {
                tagName: 'text',
                className: 'label'
            }, {
                tagName: 'text',
                className: 'close'
            }, {
                tagName: 'g',
                className: 'options',
                children: [{
                    tagName: 'g',
                    className: 'left-container',
                    children: []
                }, {
                    tagName: 'g',
                    className: 'right-container',
                    children: []
                }]
            }],
            // 这里两个数据是作为动态添加节点时，将节点添加到markup中
            markupLeftOptionJson: {
                tagName: 'g',
                children: [{
                    tagName: 'rect',
                    className: 'left-rect'
                }, {
                    tagName: 'text',
                    className: 'option-in-text'
                }]
            },
            markupRightOptionJson: {
                tagName: 'g',
                children: [{
                    tagName: 'rect',
                    className: 'right-rect'
                }, {
                    tagName: 'text',
                    className: 'option-out-text'
                }]
            },
            attrs: {
                '.body': {
                    refWidth: '100%',
                    refHeight: '100%',
                    fill: '#fff',
                    stroke: '#000000',
                },
                '.label': {
                    fill: '#000000',
                    fontSize: 18,
                    refX: .5,
                    refY: 15,
                    textAnchor: 'middle',
                    yAlignment: 'middle',
                    fontFamily: 'Arial, helvetica, sans-serif',
                    text: 'text'
                },
                '.close': {
                    text: 'X',
                    fontSize: 20,
                    fill: '#000000',
                    textAnchor: 'middle',
                    yAlignment: 'middle',
                    fontFamily: 'Arial, helvetica, sans-serif',
                    refX: .96,
                    refY: 15,
                    cursor: 'pointer',
                    event: 'element:removeRuleBox'
                },
                '.options': {
                    refY: ruleNodeStyle.titleHeight
                },
                '.right-container': {
                    refX: .5
                }
            },
            ports: {
                groups: {
                    'in': {
                        position: {
                            name: 'left',
                            args: {}
                        }
                    },
                    'out': {
                        position: {
                            name: 'right',
                            args: {}
                        }
                    }
                },
                items: []
            }
        })
    }

    /**
     * 创建node节点
     * @param index
     * @param data
     * @param paperWidth
     */
    function createNode (index, data, paperWidth) {
        let x = nodeStyle.margin.x + nodeStyle.margin.x + data.tier * nodeStyle.tierWidth, y = nodeStyle.margin.y + nodeStyle.margin.y + container.padding.y;
        if (paperWidth) {
            x = paperWidth - container.width - nodeStyle.margin.x + nodeStyle.margin.x + data.tier * nodeStyle.tierWidth;
        }
        if (index) {
            y += (nodeStyle.height + nodeStyle.margin.y) * index
        }

        class nodeModel extends joint.shapes.devs.Model {
            constructor (opt) {
                super(opt);
                this.prop('index', index);
                this.prop('tier', data.tier);
                this.attr('text/text', data.value);
                this.prop('id', data.value);
                this.prop('data', data);
                if (data.children.length) {
                    this.markup = '<g class="rotatable"><rect class="body"/><polygon /><text class="label"/></g>'
                }
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
    // function connectFn (source, target, graph) {
    // class link extends joint.dia.Link {
    //     constructor (opt) {
    //         super(opt);
    //     }
    // }
    //
    // new link({
    //     source: {
    //         id: source,
    //         port: ''
    //     },
    //     target: {
    //         id: target,
    //         port: ''
    //     },
    //     attrs: {
    //         '.connection': {
    //             'stroke-width': 1,
    //             targetMarker: {
    //                 type: 'path',
    //                 fill: '#7c68fc',
    //                 stroke: 'none',
    //                 d: 'M 20 -10 0 0 20 10 z'
    //             }
    //         }
    //         // '.marker-source': { fill: '#4b4a67', stroke: '#4b4a67', d: 'M 10 0 L 0 5 L 10 10 z' },
    //         // '.marker-target': { fill: '#4b4a67', stroke: '#4b4a67', d: 'M 10 0 L 0 5 L 10 10 z' }
    //     },
    //     markup: '<path class="connection"/><rect  class="123"/>'
    // }).addTo(graph).reparent();
    // };

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
                const rightContainer = createContainer(this.paperWidth);
                leftContainer.attr('text/text', 'HIS数据源');
                rightContainer.attr('text/text', 'FIRE数据源');
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

                        // 递归调用展开子节点
                        // if (!item.children.length)
                        //     return;
                        // return setSourceChart(item.children)
                    })
                };

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

                        // 递归调用展开子节点
                        // if (!item.children.length)
                        //     return;
                        // return setTargetChart(item.children)
                    })
                };

                setTimeout(() => {
                    setSourceChart(response.source);
                    setTargetChart(response.target);

                    this.graph.addCells(nodes);
                    // 初始化连接状态
                    // const sourceList = lodash.map(response.source, item => item.value);
                    // const targetList = lodash.map(response.target, item => item.value);
                    // lodash.forEach(connects, connect => {
                    //     if (sourceList.indexOf(connect.sourceId) !== -1 && targetList.indexOf(connect.targetId) !== -1) {
                    //         connectFn(connect.sourceId, connect.targetId, this.graph)
                    //     }
                    //     if (sourceList.indexOf(connect.targetId) !== -1 && targetList.indexOf(connect.sourceId) !== -1) {
                    //         connectFn(connect.sourceId, connect.targetId, this.graph)
                    //     }
                    // });
                    // this.graph.addCell(createRuleBox(800, 150))
                }, 100);

                this.paper.on({
                    'cell:pointerclick': this.treeClick,
                    'link:connect': this.connectFn,
                    'element:removeRuleBox': (cellView, evt) => {
                        evt.stopPropagation();
                        this.graph.removeCells([cellView.model]);
                    },
                    'tool:remove': this.disconnectFn
                    //     function (linkView, evt) {
                    //     evt.stopPropagation();
                    //     linkView.model.remove();
                    // }

                });
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
                        x = nodeStyle.margin.x + nodeStyle.margin.x + node.tier * nodeStyle.tierWidth;
                    } else {
                        x = this.paperWidth - container.width - nodeStyle.margin.x + nodeStyle.margin.x;
                    }
                    let y = nodeStyle.margin.y + nodeStyle.margin.y + container.padding.y;
                    y += (nodeStyle.height + nodeStyle.margin.y) * startIndex;
                    node.position(x, y)
                });
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
                // let otherModels;
                // if (type === 'source') {
                //     otherModels = lodash.filter(this.graph.getCells(), m => m.type === 'target');
                // } else {
                //     otherModels = lodash.filter(this.graph.getCells(), m => m.type === 'source');
                // }

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
                });

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
                    nodeItem.set('z', currentZ);
                    addNodes.push(nodeItem);
                    models.splice(startIndex, 0, nodeItem)
                });

                for (let i = startIndex + 1; i < models.length; i++) {
                    let x = 0;
                    if (type === 'source') {
                        x = nodeStyle.margin.x + nodeStyle.margin.x + models[i].tier * nodeStyle.tierWidth;
                    } else {
                        x = this.paperWidth - container.width - nodeStyle.margin.x + nodeStyle.margin.x + models[i].tier * nodeStyle.tierWidth;
                    }
                    let y = nodeStyle.margin.y + nodeStyle.margin.y + container.padding.y;
                    y += (nodeStyle.height + nodeStyle.margin.y) * i;
                    models[i].position(x, y)
                }

                this.graph.addCells(addNodes);

                // connect 展开tree查找连接节点
                // let sourceList = lodash.map(cellView.data.children, node => node.value);
                // let otherList = lodash.map(otherModels, node => node.id);
                //
                // lodash.forEach(connects, connect => {
                //     if (sourceList.indexOf(connect.sourceId) !== -1 && otherList.indexOf(connect.targetId) !== -1) {
                //         connectFn(connect.sourceId, connect.targetId, this.graph)
                //     }
                //     if (sourceList.indexOf(connect.targetId) !== -1 && otherList.indexOf(connect.sourceId) !== -1) {
                //         connectFn(connect.targetId, connect.sourceId, this.graph)
                //     }
                // })
            },
            /**
             * 画布点击列表时触发
             * @param cellView
             */
            treeClick (cellView) {
                const expand = cellView.model.get('expand');
                const type = cellView.model.get('type');
                let flag = false;

                function isConnectFn(data) {
                    const isConnect = lodash.filter(data, c => c.isConnect);
                    if (isConnect.length) {
                        flag = true
                    } else{
                        if(flag) {
                            return;
                        }
                        // 递归查询是否有节点连线
                        lodash.forEach(data, item => {
                            if(item.children.length) {
                                return isConnectFn(item.children)
                            }
                        });
                    }
                }

                if ((type !== 'source' && type !== 'target') || !cellView.model.data || !cellView.model.data.children.length)
                    return;

                if (expand) {
                    // 如果有子节点处于连接状态 则不能关闭tree
                    isConnectFn(cellView.model.data.children);
                    if (flag) {
                        return;
                    }
                    this.operationCloseTree(type, cellView.model, cellView.model.collection.models);
                    cellView.model.set('expand', false);
                    cellView.model.expandFn(false);
                } else {
                    this.operationExpandTree(type, cellView.model, cellView.model.collection.models);
                    cellView.model.set('expand', true);
                    cellView.model.expandFn(true);
                }
            },
            /**
             * 连接时触发
             * @param linkView
             */
            connectFn (linkView) {
                //do something
                this.setIsConnect(linkView.model.getSourceElement(), true);
                this.setIsConnect(linkView.model.getTargetElement(), true);
            },
            setIsConnect (model, boole) {
                const type = model.get('type');
                if (type === 'source' || type === 'target') {
                    model.data.isConnect = boole;
                }
            },
            disconnectFn (linkView, evt) {
                evt.stopPropagation();
                this.setIsConnect(linkView.model.getSourceElement(), false);
                this.setIsConnect(linkView.model.getTargetElement(), false);
                linkView.model.remove();
            },
            drop (evt) {
                const text = evt.dataTransfer.getData("value");
                const position = {
                    x: evt.layerX,
                    y: evt.layerY
                };
                const rule = ruleMap[text];
                const node = createRuleBox(position, rule);
                node.attr('.label/text', text);
                node.set('idInfo', text);
                this.graph.addCell(node);
                this.addElements(node, rule);

            },
            drag (evt) {
                evt.dataTransfer.setData("value", evt.target.innerHTML);
            },
            allowDrop (ev) {
                ev.preventDefault();
            },
            addElements (node, rule) {
                const inPort = {
                    group: 'in',
                    args: {
                        y: 45
                    },
                    markup: '<circle class="port-body joint-port-body" r="10" fill="gray" magnet="true"/>'
                };
                const outPort = {
                    group: 'out',
                    args: {
                        y: 45
                    },
                    markup: '<circle class="port-body joint-port-body" r="10" fill="gray" magnet="true"/>'
                };

                let maxHeight = 0;
                // 计算容器宽高 以及 元素所在位置
                if (rule.inPorts.length !== rule.outPorts.length) {
                    if (rule.inPorts.length > rule.outPorts.length) {
                        maxHeight = ruleNodeStyle.itemHeight * rule.inPorts.length;
                        node.attr('.left-rect', {
                            height: ruleNodeStyle.itemHeight
                        });
                        node.attr('.right-rect', {
                            height: maxHeight
                        });
                        node.attr('.option-in-text', {
                            refY: ruleNodeStyle.itemHeight / 2
                        });
                        node.attr('.option-out-text', {
                            refY: maxHeight / 2
                        });
                        outPort.args.y = maxHeight / 2 + ruleNodeStyle.titleHeight
                    } else {
                        maxHeight = ruleNodeStyle.itemHeight * rule.outPorts.length;
                        node.attr('.left-rect', {
                            height: maxHeight
                        });
                        node.attr('.right-rect', {
                            height: ruleNodeStyle.itemHeight
                        });
                        node.attr('.option-in-text', {
                            refY: maxHeight / 2
                        });
                        node.attr('.option-out-text', {
                            refY: ruleNodeStyle.itemHeight / 2
                        });
                        inPort.args.y = maxHeight / 2 + ruleNodeStyle.titleHeight
                    }
                } else {
                    node.attr('.left-rect', {
                        height: ruleNodeStyle.itemHeight
                    });
                    node.attr('.right-rect', {
                        height: ruleNodeStyle.itemHeight
                    });
                    node.attr('.option-in-text', {
                        refY: ruleNodeStyle.itemHeight / 2
                    });
                    node.attr('.option-out-text', {
                        refY: ruleNodeStyle.itemHeight / 2
                    });
                }
                ruleNodeStyle.height = maxHeight + ruleNodeStyle.titleHeight;

                // 设置样式
                node.attr('.left-rect', {
                    stroke: '#000',
                    strokeWidth: 1,
                    strokeOpacity: .8,
                    fill: 'white',
                    width: ruleNodeStyle.width / 2
                });
                node.attr('.right-rect', {
                    stroke: '#000',
                    strokeWidth: 1,
                    strokeOpacity: .8,
                    fill: 'white',
                    width: ruleNodeStyle.width / 2
                });
                node.attr('.option-in-text', {
                    fill: '#000000',
                    fontSize: 12,
                    fontFamily: 'Arial, helvetica, sans-serif',
                    text: 'value1',
                    yAlignment: 'middle',
                    refX: 20
                });
                node.attr('.option-out-text', {
                    fill: '#000000',
                    fontSize: 12,
                    fontFamily: 'Arial, helvetica, sans-serif',
                    text: 'value2',
                    yAlignment: 'middle',
                    refX: 20
                });

                // 获取节点视图
                const nodeView = this.paper.findViewByModel(node);
                // 将字符换为svg元素
                const leftContainer = nodeView.$el.find('.options .left-container');
                const rightContainer = nodeView.$el.find('.options .right-container');
                const markup = node.get('markup')

                // 循环添加元素和port
                lodash.forEach(rule.inPorts, (item, index) => {
                    const markupLeftOption = joint.V(node.get('markupLeftOption'));
                    markupLeftOption.addClass('option' + index);
                    leftContainer.append(markupLeftOption.node);
                    node.addPort(inPort);
                    inPort.args.y += ruleNodeStyle.itemHeight;
                    node.attr(`.left-container .option${index}`, {
                        refY: index * ruleNodeStyle.itemHeight
                    });
                    node.attr(`.left-container .option${index} text/text`, item);

                    // 更新markup 通过optionJson
                    const markupLeftOptionJson = lodash.cloneDeep(node.get('markupLeftOptionJson'));
                    lodash.forEach(markup, m => {
                        if (m.className === 'options') {
                            markupLeftOptionJson.className = 'option' + index;
                            m.children[0].children.push(markupLeftOptionJson);
                        }
                    })

                });
                lodash.forEach(rule.outPorts, (item, index) => {
                    const markupRightOption = joint.V(node.get('markupRightOption'));
                    markupRightOption.addClass('option' + index);
                    rightContainer.append(markupRightOption.node);
                    node.addPort(outPort);
                    outPort.args.y += ruleNodeStyle.itemHeight;
                    node.attr(`.right-container .option${index}`, {
                        refY: index * ruleNodeStyle.itemHeight
                    });
                    node.attr(`.right-container .option${index} text/text`, item);

                    // 更新markup 通过optionJson
                    const markupRightOptionJson = lodash.cloneDeep(node.get('markupRightOptionJson'));
                    lodash.forEach(markup, m => {
                        if (m.className === 'options') {
                            markupRightOptionJson.className = 'option' + index;
                            m.children[1].children.push(markupRightOptionJson);
                        }
                    })
                });

                // 设置计算后规则容器大小
                node.prop('size', {
                    width: ruleNodeStyle.width,
                    height: ruleNodeStyle.height
                });
            },
            save () {
                window.cells = this.graph.getCells()
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
