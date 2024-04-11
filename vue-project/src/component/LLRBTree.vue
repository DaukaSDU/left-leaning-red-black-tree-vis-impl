<script setup lang="ts">
import { reactive, ref } from "vue"
import * as vNG from "v-network-graph"

// implementation of Node class
class Node {
  data: number;
  isRed: boolean;
  left: Node;
  right: Node;

  constructor(data) {
    this.data = data;
    this.isRed = true;
    this.left = this.right = null;
  }
}

// implementation of LeftLeaningRedBlackTree class
class Tree {
  root: Node
  countOperation: number
  operations: Node[]

  constructor() {
    this.root = null
  }

  rotateLeft(h:Node) {
    const x = h.right
    h.right = x.left
    x.left = h
    x.isRed = h.isRed
    h.isRed = true
    this.countOperation++
    return x
  }

  rotateRight(h:Node) {
    const x = h.left
    h.left = x.right
    x.right = h
    x.isRed = h.isRed
    h.isRed = true
    this.countOperation++
    return x
  }

  flipColors(h:Node) {
    h.isRed = !h.isRed
    h.left.isRed = !h.left.isRed
    h.right.isRed = !h.right.isRed
    this.countOperation++
  }

  insertNode(data:number) {
    this.countOperation = 0
    this.operations = []
    this.root = this.insert(this.root, data)
    this.root.isRed = false
  }

  insert(h:Node, data:number) {
    let rotatedRight = false

    if (h == null) return new Node(data)

    if (data < h.data)
        h.left = this.insert(h.left, data)
    else if (data > h.data)
        h.right = this.insert(h.right, data)

    if (this.isRed(h.right) && !this.isRed(h.left)) {
        const clone = this.clone()
        this.operations.push(clone)
        h = this.rotateLeft(h)
    }
    if (this.isRed(h.left) && this.isRed(h.left.left)) {
        const clone = this.clone()
        this.operations.push(clone)
        h = this.rotateRight(h)
        rotatedRight = true
    }
    if (this.isRed(h.left) && this.isRed(h.right)) {
        let clone = this.clone()
        if (rotatedRight) {
          let temp = clone
          if (clone.data == h.right.data) clone = this.makeClone(h)
          else {
            while (temp != null) {
                if (temp.left != null && h.right.data < temp.data) {
                    if (h.right.data != temp.left.data) temp = temp.left
                    else {temp.left = this.makeClone(h); break;}                  
                }
                if (temp.right != null && h.right.data > temp.data) {                  
                    if (h.right.data != temp.right.data) temp = temp.right
                    else {temp.right = this.makeClone(h); break;}                  
                }
            }
          }
        }
        this.operations.push(clone);
        this.flipColors(h);
    }

    return h;
  }

  isRed(x:Node) {
      if (x == null)
          return false
      return x.isRed
  }

  clone() {
    return this.makeClone(this.root)
  }

  makeClone(root:Node) {
    if (root == null) return null

    const clone = new Node(root.data)
    clone.isRed = root.isRed

    if (root.left != null) {
      clone.left = this.makeClone(root.left)
    } 
    if (root.right != null) {
      clone.right = this.makeClone(root.right)
    }
    return clone;
  }
}

// configs
const configs = reactive(
  vNG.defineConfigs({
    view: {
      scalingObjects: true,
      minZoomLevel: 0.1,
      maxZoomLevel: 16,
    },
    node: {
        label: {
            color: "#000000",
            margin: 4,
            direction: "center",
            text: "name",
        },
        normal: {
            color: "#fff",
            strokeWidth: 1,
            strokeColor: "#000",
            radius: 20,
        },
        hover: {
            color: "#fff",
            radius: 20,
            strokeWidth: 2,
        },
        transition: "all 1s linear"
    },
    edge: {
        normal: {
            color: "#000",
            width: 1,
        },
        hover: {
            color: "#000",
            width: 2,
        }
    },
    path: {
        visible: true,
        normal: {
          width: 3,
          color: "red",
        },
        margin: 21,
    },
  })
)
const zoomLevel = ref(1.5)

// reactive elements
let new_val = ref()
const nodes = ref([])
const edges = ref([])
const layouts = ref({ nodes: nodes })
const paths = ref([])

// inserting a new node
function add() {
  tree.insertNode(Number(new_val.value))
  new_val.value = null
  console.log(tree.operations)
  showOperations()
  setTimeout(() => draw(tree.root), 1000 * tree.countOperation)
}

// showing every operation when it inserts a new node
function showOperations() {
  let i = 0
  for (let operation of tree.operations) {
    setTimeout(() => draw(operation), 1000 * i++)
  }
}

// drawing tree
function draw(root:Node) {
  const height = Number(Math.log2(nodes.value.length + 1))
  nodes.value = []
  edges.value = []
  paths.value = []
  drawTree(root, -1, 0, -200, 50 * height * 2)
  /* drawTree(root, parentID, position X, position Y, width) */
}

//drawing tree
function drawTree(root:Node, source:number, x:number, y:number, width) {
  if (root == null) return
  width /= 2
  const target = nodes.value.length
  nodes.value.push({ name: `${root.data}`, x: x, y: y})
  edges.value.push({ source: source, target: target})
  if (root.isRed) paths.value.push({ edges: [target] })
  drawTree(root.left, target, x - width, y + 50, width)
  drawTree(root.right, target, x + width, y + 50, width)
}

const tree = new Tree();
</script>

<template>
  <div>
    Enter: <input v-model="new_val" type="text" id="new_val">
    <button @click="add()">add</button>
  </div>

  <div class="demo-control-panel">
    <el-checkbox v-model="configs.view.scalingObjects">Scaling objects</el-checkbox>
    <el-slider-custom-zoom v-model="zoomLevel" />
  </div>

    <v-network-graph
    v-model:zoom-level="zoomLevel"
    :nodes="nodes"
    :edges="edges"
    :layouts="layouts"
    :configs="configs"
    :paths="paths"
    style="height: 100vh;"
    />
</template>