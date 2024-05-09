<template>
    <div id="app">
        <h1 style="text-align: center">Vue Grid Layout</h1>
        <!--<pre>{{ layout | json }}</pre>-->
        <div>
            <div class="layoutJSON">
                Displayed as <code>[x, y, w, h]</code>:
                <div class="columns">
                    <div class="layoutItem" v-for="item in layout" :key="item.i">
                        <b>{{item.i}}</b>: [{{item.x}}, {{item.y}}, {{item.w}}, {{item.h}}]
                    </div>
                </div>
            </div>
            <!--<div class="layoutJSON">
                Displayed as <code>[x, y, w, h]</code>:
                <div class="columns">
                    <div class="layoutItem" v-for="item in layout2">
                        <b>{{item.i}}</b>: [{{item.x}}, {{item.y}}, {{item.w}}, {{item.h}}]
                    </div>
                </div>
            </div>-->
        </div>
        <div id="content">
            <button @click="decreaseWidth">Decrease Width</button>
            <button @click="increaseWidth">Increase Width</button>
            <button @click="addItem">Add an item</button>
            <button @click="addItemDynamically">Add an item dynamically</button>
            <!-- Add to show rtl support -->
            <button @click="changeDirection">Change Direction</button>
            <input type="checkbox" v-model="draggable"/> Draggable
            <input type="checkbox" v-model="bounded"/> Bounded
            <input type="checkbox" v-model="resizable"/> Resizable
            <input type="checkbox" v-model="mirrored"/> Mirrored
            <input type="checkbox" v-model="responsive"/> Responsive
            <input type="checkbox" v-model="preventCollision"/> Prevent Collision
            <div style="margin-top: 10px;margin-bottom: 10px;">
                Row Height: <input type="number" v-model="rowHeight"/> Col nums: <input type="number" v-model="colNum"/>
                Margin x: <input type="number" v-model="marginX"/> Margin y: <input type="number" v-model="marginY"/>
            </div>
            <grid-layout
                :margin="[parseInt(marginX), parseInt(marginY)]"
                v-model:layout="layout"
                :responsive-layouts="layouts"
                :col-num="parseInt(colNum)"
                :row-height="rowHeight"
                :is-draggable="draggable"
                :is-resizable="resizable"
                :is-mirrored="mirrored"
                :is-bounded="bounded"
                :prevent-collision="preventCollision"
                :vertical-compact="compact"
                :use-css-transforms="true"
                :responsive="responsive"
                :max-rows="15"
                ref="gridlayout"
                @layout-created="layoutCreatedEvent"
                @layout-before-mount="layoutBeforeMountEvent"
                @layout-mounted="layoutMountedEvent"
                @layout-ready="layoutReadyEvent"
                @layout-updated="layoutUpdatedEvent"
                @breakpoint-changed="breakpointChangedEvent"
            >
                <grid-item v-for="item in layout"
                           :key="item.i"
                           :static="item.static"
                           :x="item.x"
                           :y="item.y"
                           :w="item.w"
                           :h="item.h"
                           :i="item.i"
                           :min-w="item.minW"
                           :max-w="item.maxW"
                           :min-x="item.minX"
                           :max-x="item.maxX"
                           :min-y="item.minY"
                           :max-y="item.maxY"
                           :preserve-aspect-ratio="item.preserveAspectRatio"
                           :ref="setItemRefArr"
                           @resize="resize"
                           @move="move"
                           @resized="resized"
                           @container-resized="containerResized"
                           @moved="moved"
                           @mouseup="e => mouseup(e, item)"
                           @moveing="mouseXYChange"
                           @movestart="movestart"
                >
                    <!--<custom-drag-element :text="item.i"></custom-drag-element>-->
                    <test-element :text="item.i" @removeItem="removeItem($event)"></test-element>
                    <!--<button @click="clicked">CLICK ME!</button>-->
                </grid-item>
            </grid-layout>
            <hr/>
        </div>
    </div>
</template>

<script>
    import GridItem from './components/GridItem.vue';
    import GridLayout from './components/GridLayout.vue';
    import TestElement from './components/TestElement.vue';
    // import CustomDragElement from './components/CustomDragElement.vue';
    import {getDocumentDir, setDocumentDir} from "./helpers/DOM";

    var testLayouts = {
        md: [
            {"x":0, "y":0, "w":2, "h":2, "i":"0"},
            {"x":2, "y":0, "w":2, "h":4, "i":"1"},
            {"x":4, "y":0, "w":2, "h":5, "i":"2"},
            {"x":6, "y":0, "w":2, "h":3, "i":"3"},
            {"x":2, "y":4, "w":2, "h":3, "i":"4"},
            {"x":4, "y":5, "w":2, "h":3, "i":"5"},
            {"x":0, "y":2, "w":2, "h":5, "i":"6"},
            {"x":2, "y":7, "w":2, "h":5, "i":"7"},
            {"x":4, "y":8, "w":2, "h":5, "i":"8"},
            {"x":6, "y":3, "w":2, "h":4, "i":"9"},
            {"x":0, "y":7, "w":2, "h":4, "i":"10"},
            {"x":2, "y":19, "w":2, "h":4, "i":"11"},
            {"x":0, "y":14, "w":2, "h":5, "i":"12"},
            {"x":2, "y":14, "w":2, "h":5, "i":"13"},
            {"x":4, "y":13, "w":2, "h":4, "i":"14"},
            {"x":6, "y":7, "w":2, "h":4, "i":"15"},
            {"x":0, "y":19, "w":2, "h":5, "i":"16"},
            {"x":8, "y":0, "w":2, "h":2, "i":"17"},
            {"x":0, "y":11, "w":2, "h":3, "i":"18"},
            {"x":2, "y":12, "w":2, "h":2, "i":"19"}
        ],
        lg: [
            {"x":0,"y":0,"w":2,"h":2,"i":"0"},
            {"x":2,"y":0,"w":2,"h":4,"i":"1"},
            {"x":4,"y":0,"w":2,"h":5,"i":"2"},
            {"x":6,"y":0,"w":2,"h":3,"i":"3"},
            {"x":8,"y":0,"w":2,"h":3,"i":"4"},
            {"x":10,"y":0,"w":2,"h":3,"i":"5"},
            {"x":0,"y":5,"w":2,"h":5,"i":"6"},
            {"x":2,"y":5,"w":2,"h":5,"i":"7"},
            {"x":4,"y":5,"w":2,"h":5,"i":"8"},
            // {"x":6,"y":4,"w":2,"h":4,"i":"9"},
            // {"x":8,"y":4,"w":2,"h":4,"i":"10"},
            // {"x":10,"y":4,"w":2,"h":4,"i":"11"},
            // {"x":0,"y":10,"w":2,"h":5,"i":"12"},
            // {"x":2,"y":10,"w":2,"h":5,"i":"13"},
            // {"x":4,"y":8,"w":2,"h":4,"i":"14"},
            // {"x":6,"y":8,"w":2,"h":4,"i":"15"},
            // {"x":8,"y":10,"w":2,"h":5,"i":"16"},
            // {"x":10,"y":4,"w":2,"h":2,"i":"17"},
            // {"x":0,"y":9,"w":2,"h":3,"i":"18"},
            // {"x":2,"y":6,"w":2,"h":2,"i":"19"}
        ],
    };
    let mouseXY = {"x": null, "y": null};
    let DragPos = {"x": null, "y": null, "w": 1, "h": 1, "i": null};

    /*let testLayout = [
        {"x":0,"y":0,"w":2,"h":2,"i":"0", resizable: true, draggable: true, static: false, minY: 0, maxY: 2},
        {"x":2,"y":0,"w":2,"h":4,"i":"1", resizable: null, draggable: null, static: true},
        {"x":4,"y":0,"w":2,"h":5,"i":"2", resizable: false, draggable: false, static: false, minX: 4, maxX: 4, minW: 2, maxW: 2, preserveAspectRatio: true},
        {"x":6,"y":0,"w":2,"h":3,"i":"3", resizable: false, draggable: false, static: false},
        {"x":8,"y":0,"w":2,"h":3,"i":"4", resizable: false, draggable: false, static: false},
        {"x":10,"y":0,"w":2,"h":3,"i":"5", resizable: false, draggable: false, static: false},
        {"x":0,"y":5,"w":2,"h":5,"i":"6", resizable: false, draggable: false, static: false},
        {"x":2,"y":5,"w":2,"h":5,"i":"7", resizable: false, draggable: false, static: false},
        {"x":4,"y":5,"w":2,"h":5,"i":"8", resizable: false, draggable: false, static: false},
        {"x":6,"y":3,"w":2,"h":4,"i":"9", resizable: false, draggable: false, static: true},
        {"x":8,"y":4,"w":2,"h":4,"i":"10", resizable: false, draggable: false, static: false},
        {"x":10,"y":4,"w":2,"h":4,"i":"11", resizable: false, draggable: false, static: false, minY: 4},
        {"x":0,"y":10,"w":2,"h":5,"i":"12", resizable: false, draggable: false, static: false},
        {"x":2,"y":10,"w":2,"h":5,"i":"13", resizable: false, draggable: false, static: false},
        {"x":4,"y":8,"w":2,"h":4,"i":"14", resizable: false, draggable: false, static: false},
        {"x":6,"y":8,"w":2,"h":4,"i":"15", resizable: false, draggable: false, static: false},
        {"x":8,"y":10,"w":2,"h":5,"i":"16", resizable: false, draggable: false, static: false},
        {"x":10,"y":4,"w":2,"h":2,"i":"17", resizable: false, draggable: false, static: false},
        {"x":0,"y":9,"w":2,"h":3,"i":"18", resizable: false, draggable: false, static: false},
        {"x":2,"y":6,"w":2,"h":2,"i":"19", resizable: false, draggable: false, static: false}
    ];*/

    export default {
        name: 'app',
        components: {
            GridLayout,
            GridItem,
            TestElement,
            // CustomDragElement,
        },
        data () {
            return {
                // layout: JSON.parse(JSON.stringify(testLayout)),
                // layout2: JSON.parse(JSON.stringify(testLayout)),
                layouts: testLayouts,
                layout: testLayouts["lg"],
                layouts1: JSON.parse(JSON.stringify(testLayouts)),
                layout1: JSON.parse(JSON.stringify(testLayouts["lg"])),
                draggable: true,
                resizable: true,
                mirrored: false,
                bounded: false,
                responsive: false,
                preventCollision: true,
                compact: true,
                rowHeight: 30,
                colNum: 12,
                index: 0,
                marginX: 10,
                marginY: 10,
                itemRefArr: [],
                moveObj: null,
            }
        },
        mounted: function () {
          let that = this;
          document.addEventListener("dragover", function (e) {
              that.mouseXYChange(e);
          }, false);
            this.index = this.layout.length;
        },
        methods: {
          movestart({i, event}) {
            this.moveObj = {i}
          },
          setItemRefArr(el) {
            const flag = this.itemRefArr.some((item) => (item && el && (item.i === el.i)))
            if (el && !flag) {
              this.itemRefArr.push(el)
            }
          },
          mouseup(e, data) {
            // console.log(this.itemRefArr, '+++++++++++itemRefArr++++++++++');
            const mouseGrid = this.itemRefArr.find(item => {
                const curRect = item.$el.getBoundingClientRect();
                // console.log(item, '+++++++++++curRect++++++++++', curRect);
                return mouseXY.x && mouseXY.y && this.moveObj && this.moveObj.i && item.i !== this.moveObj.i && ((mouseXY.x > curRect.left) && (mouseXY.x < curRect.right)) && ((mouseXY.y > curRect.top) && (mouseXY.y < curRect.bottom))
            });
            if(mouseGrid) {
              console.log(mouseGrid, '+++++++++++mouseup++++++++++', mouseXY);
              // this.itemRefArr = this.itemRefArr.filter(obj => obj.i !== this.moveObj.i && mouseGrid.i !== obj.i);
              this.layout = this.layout.filter(obj => obj.i !== this.moveObj.i && mouseGrid.i !== obj.i);
              // console.log(this.itemRefArr, e, '+++++++++++mouseup++++++++++', data, items);
            }
            this.moveObj = null;
          },
          mouseXYChange: function(e) {
            mouseXY.x = e.clientX;
            mouseXY.y = e.clientY;
            // console.log(e, '======mouseXYChange======', mouseXY);
          },
          drag: function (e) {
            let parentRect = document.getElementById('content').getBoundingClientRect();
            let mouseInGrid = false;
            if (((mouseXY.x > parentRect.left) && (mouseXY.x < parentRect.right)) && ((mouseXY.y > parentRect.top) && (mouseXY.y < parentRect.bottom))) {
                mouseInGrid = true;
            }
            if (mouseInGrid === true && (this.layout.findIndex(item => item.i === 'drop')) === -1) {
                this.layout.push({
                    x: (this.layout.length * 2) % (this.colNum || 12),
                    y: this.layout.length + (this.colNum || 12), // puts it at the bottom
                    w: 1,
                    h: 1,
                    i: 'drop',
                });
            }
            let index = this.layout.findIndex(item => item.i === 'drop');
            if (index !== -1) {
            //   // console.log(this.$refs.gridlayout, '++++++1122334++++', this.layout.length);
            //   //   try {
            //   //       this.$refs.gridlayout.$el.children[this.layout.length].$refs.item.style.display = "none";
            //   //   } catch {
            //   //     console.log('111111')
            //   //   }
                let el = this.$refs['gridItemdrop'];
                el.dragging = {"top": mouseXY.y - parentRect.top, "left": mouseXY.x - parentRect.left};
                // let new_pos = el.calcXY(mouseXY.y - parentRect.top, mouseXY.x - parentRect.left);

            //     if (mouseInGrid === true) {
            //         this.$refs.gridlayout.dragEvent('dragstart', 'drop', new_pos.x, new_pos.y, 1, 1);
            //         DragPos.i = String(index);
            //         DragPos.x = this.layout[index].x;
            //         DragPos.y = this.layout[index].y;
            //     }
            //     if (mouseInGrid === false) {
            //         this.$refs.gridlayout.dragEvent('dragend', 'drop', new_pos.x, new_pos.y, 1, 1);
            //         this.layout = this.layout.filter(obj => obj.i !== 'drop');
            //     }
              console.log(this.$refs, '============', el)
            }
        },
        dragend: function () {
            let parentRect = document.getElementById('content').getBoundingClientRect();
            let mouseInGrid = false;
            if (((mouseXY.x > parentRect.left) && (mouseXY.x < parentRect.right)) && ((mouseXY.y > parentRect.top) && (mouseXY.y < parentRect.bottom))) {
                mouseInGrid = true;
            }
            if (mouseInGrid === true) {
                // alert(`Dropped element props:\n${JSON.stringify(DragPos, ['x', 'y', 'w', 'h'], 2)}`);
                this.$refs.gridlayout.dragEvent('dragend', 'drop', DragPos.x, DragPos.y, 1, 1);
                this.layout = this.layout.filter(obj => obj.i !== 'drop');

                // UNCOMMENT below if you want to add a grid-item
                /*
                this.layout.push({
                    x: DragPos.x,
                    y: DragPos.y,
                    w: 1,
                    h: 1,
                    i: DragPos.i,
                });
                this.$refs.gridLayout.dragEvent('dragend', DragPos.i, DragPos.x,DragPos.y,1,1);
                try {
                    this.$refs.gridLayout.$children[this.layout.length].$refs.item.style.display="block";
                } catch {
                }
                */
            }
        },
            clicked: function() {
                window.alert("CLICK!");
            },
            increaseWidth: function() {
                let width = document.getElementById("content").offsetWidth;
                width += 20;
                document.getElementById("content").style.width = width+"px";
            },
            decreaseWidth: function() {
                let width = document.getElementById("content").offsetWidth;
                width -= 20;
                document.getElementById("content").style.width = width+"px";
            },
            removeItem: function(i) {
                console.log("### REMOVE " + i);
                const index = this.layout.map(item => item.i).indexOf(i);
                this.layout.splice(index, 1);
            },
            addItem: function() {
                // let self = this;
                //console.log("### LENGTH: " + this.layout.length);
                let item = {"x":0,"y":0,"w":2,"h":2,"i":this.index+"", whatever: "bbb"};
                this.index++;
                this.layout.push(item);
            },
            addItemDynamically: function() {
                const x = (this.layout.length * 2) % (this.colNum || 12);
                const y = this.layout.length + (this.colNum || 12);
                console.log("X=" + x + " Y=" + y)
                let item = {
                  x: x,
                  y: y,
                  w: 2,
                  h: 2,
                  i: this.index+"",
                }
                this.index++;
                this.layout.push(item);
            },
            move: function(i, newX, newY){
                // console.log("MOVE i=" + i + ", X=" + newX + ", Y=" + newY);
            },
            resize: function(i, newH, newW, newHPx, newWPx){
                // console.log("RESIZE i=" + i + ", H=" + newH + ", W=" + newW + ", H(px)=" + newHPx + ", W(px)=" + newWPx);
            },
            moved: function(i, newX, newY){
                // console.log("### MOVED i=" + i + ", X=" + newX + ", Y=" + newY);
            },
            resized: function(i, newH, newW, newHPx, newWPx){
                // console.log("### RESIZED i=" + i + ", H=" + newH + ", W=" + newW + ", H(px)=" + newHPx + ", W(px)=" + newWPx);
            },
            containerResized: function(i, newH, newW, newHPx, newWPx){
                // console.log("### CONTAINER RESIZED i=" + i + ", H=" + newH + ", W=" + newW + ", H(px)=" + newHPx + ", W(px)=" + newWPx);
            },
            /**
             * Add change direction button
             */
            changeDirection() {
                let documentDirection = getDocumentDir();
                let toggle = "";
                if (documentDirection === "rtl") {
                    toggle = "ltr"
                } else {
                    toggle = "rtl"
                }
                setDocumentDir(toggle);
                //eventBus.$emit('directionchange');
            },

            layoutCreatedEvent: function(newLayout){
                // console.log("Created layout: ", newLayout)
            },
            layoutBeforeMountEvent: function(newLayout){
                // console.log("beforeMount layout: ", newLayout)
            },
            layoutMountedEvent: function(newLayout){
                // console.log("Mounted layout: ", newLayout)
            },
            layoutReadyEvent: function(newLayout){
                // console.log("Ready layout: ", newLayout)
                this.layout = newLayout
            },
            layoutUpdatedEvent: function(newLayout){
                // console.log("Updated layout: ", newLayout)
                this.layout = newLayout
            },
            layoutReadyEvent1: function(newLayout){
                // console.log("Ready layout: ", newLayout)
                this.layout1 = newLayout
            },
            layoutUpdatedEvent1: function(newLayout){
                // console.log("Updated layout: ", newLayout)
                this.layout1 = newLayout
            },
            breakpointChangedEvent: function(newBreakpoint, newLayout){
                // console.log("breakpoint changed breakpoint=", newBreakpoint, ", layout: ", newLayout );
            }

        },
    }
</script>

<style>
    /*    #app {
            font-family: 'Avenir', Helvetica, Arial, sans-serif;
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
            text-align: center;
            color: #2c3e50;
            margin-top: 60px;
        }

        h1, h2 {
            font-weight: normal;
        }

        ul {
            list-style-type: none;
            padding: 0;
        }

        li {
            display: inline-block;
            margin: 0 10px;
        }

        a {
            color: #42b983;
        }*/
</style>

<style lang="scss">
.droppable-element {
    width: 150px;
    text-align: center;
    background: #fdd;
    border: 1px solid black;
    margin: 10px 0;
    padding: 10px;
}
.vue-grid-layout {
    background: #eee;
}

.vue-grid-item:not(.vue-grid-placeholder) {
    background: #ccc;
    border: 1px solid black;
}

.vue-grid-item .resizing {
    opacity: 0.9;
}

.vue-grid-item .static {
    background: #cce;
}

.vue-grid-item .text {
    font-size: 24px;
    text-align: center;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
    height: 100%;
    width: 100%;
}

.vue-grid-item .no-drag {
    height: 100%;
    width: 100%;
}

.vue-grid-item .minMax {
    font-size: 12px;
}

.vue-grid-item .add {
    cursor: pointer;
}

.vue-draggable-handle {
    position: absolute;
    width: 20px;
    height: 20px;
    top: 0;
    left: 0;
    background: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='10'><circle cx='5' cy='5' r='5' fill='#999999'/></svg>") no-repeat;
    background-position: bottom right;
    padding: 0 8px 8px 0;
    background-repeat: no-repeat;
    background-origin: content-box;
    box-sizing: border-box;
    cursor: pointer;
}

.layoutJSON {
    background: #ddd;
    border: 1px solid black;
    margin-top: 10px;
    padding: 10px;
}

.layoutJSON {
    background: #ddd;
    border: 1px solid black;
    margin-top: 10px;
    padding: 10px;
}

.columns {
    -moz-columns: 120px;
    -webkit-columns: 120px;
    columns: 120px;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /*text-align: center;*/
  color: #2c3e50;
  /*margin-top: 60px;*/
}
</style>
