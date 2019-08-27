算法 
---
### 目录
- <a href="#sort">排序</a>
  -   <a href="#sort1">冒泡排序</a>
  -   <a href="#sort2">选择排序</a>
  -   <a href="#sort3">插入排序</a>
  -   <a href="#sort4">合并排序</a>
  -   <a href="#sort5">快速排序</a>
- <a href="#binaryTree">二叉树</a>
- <a href="#depthTraver">深度优先遍历</a>
- <a href="#dreadthTraver">广度优先遍历</a>
- <a href="#maxSum">求数组最大连续子集和</a>
- <a href="#arrToTree">list转成树结构</a>
- <a href="#deepCopy">深度拷贝</a>

### <a name="sort">排序</a>
1.<a name="sort1">冒泡排序</a>
```javascript
const bubbleSort = (arr=[]) => {
    let len = arr.length;
    for(let i=0;i<len-1;i++) {
        for(let j=i+1;j<len;j++) {
            let cur = arr[i];
            let next = arr[j];
            let temp = 0;
            if(cur>next) {
                temp = cur;
                arr[i] = next;
                arr[j] = temp;
            }
        }
    }
    return arr
};
let bubbleSortResult = bubbleSort([3,5,1,4,2,7,6,9,8,12,11,13]);
console.log('bubbleSortResult==>', bubbleSortResult);
```
2.<a name="sort2">选择排序</a>
```javascript
const selectionSort = (arr=[]) => {
    let len = arr.length;
    for(let i=0; i<len-1; i++) {
        let min = arr[i];
        let minIndex = 0;
        let temp = 0;
        for(let j=i+1; j<len; j++) {
            let next = arr[j];
            if(min>next) {
                min = next;
                minIndex = j
            }
        }
        if(arr[i] !== min) {
            temp = arr[i];
            arr[i] = min;
            arr[minIndex] = temp
        }
    }
    return arr
};
let selectionSortResult = selectionSort([3,5,1,4,2,7,6,9,8,12,11,13]);
console.log('selectionSortResult==>', selectionSortResult);
```
3.<a name="sort3">插入排序</a>
```javascript
const insertionSort = (arr=[]) => {
    let len = arr.length;
    let cur,
        i,
        j;
    for(i=0; i<len; i++) {
        cur = arr[i];
        for(j=i-1; j>-1 && arr[j]>cur; j--) {
            arr[j+1] = arr[j]
        }
        arr[j+1] = cur
    }
    return arr
};
let insertionSortResult = insertionSort([3,5,1,4,2,7,6,9,8,12,11,13]);
console.log('insertionSortResult==>', insertionSortResult);
```
4.<a name="sort4">合并排序</a>
```javascript
function merge(left, right){
    let result  = [],
        il      = 0,
        ir      = 0;

    while (il < left.length && ir < right.length){
        if (left[il] < right[ir]){
            result.push(left[il++]);
        } else {
            result.push(right[ir++]);
        }
    }

    return result.concat(left.slice(il)).concat(right.slice(ir));
}
function mergeSort(myArray){
    if (myArray.length < 2) {
        return myArray;
    }

    let middle = Math.floor(myArray.length / 2),
        left    = myArray.slice(0, middle),
        right   = myArray.slice(middle);

    return merge(mergeSort(left), mergeSort(right));
}
let mergeSortResult = mergeSort([3,5,1]);
console.log('mergeSortResult==>', mergeSortResult);
```
5.<a name="sort5">快速排序</a>
```javascript
const quickSort = (arr=[]) => {
    let len = arr.length;
    if(len<2) return arr;
    let pivot = arr.splice([Math.floor(len/2)],1)[0];
    let left = [];
    let right = [];
    for(let i=0; i<arr.length; i++) {
        if(arr[i]>pivot) {
            right.push(arr[i])
        }else {
            left.push(arr[i])
        }
    }
    return quickSort(left).concat([pivot], quickSort(right))
};
let quickSortResult = quickSort([3,5,1]);
console.log('quickSortResult==>', quickSortResult);
```
### <a name="binaryTree">二叉树</a>
```javascript
function BinaryTree() {
    let root = null;

    function Node(key) {
        this.key = key;
        this.left = null;
        this.right = null;
    }

    function insertNode(node, newNode) {
        if (newNode.key < node) {
            if (node.left === null) {
                node.left = newNode
            } else {
                insertNode(node.left, newNode)
            }
        } else {
            if (node.right === null) {
                node.right = newNode
            } else {
                insertNode(node.right, newNode)
            }
        }
    }

    // 生成二叉树
    this.insert = function (key) {
        let newNode = new Node(key);
        if (root === null) {
            root = newNode
        } else {
            insertNode(root, newNode)
        }
    };

    // 二叉树-前序遍历（复制二叉树）
    function preOrderTravelNode(root, callback) {
        if (root !== null) {
            callback(root.key);
            preOrderTravelNode(root.left, callback);
            preOrderTravelNode(root.right, callback);
        }
    }

    this.preOrderTravel = function (callback) {
        preOrderTravelNode(root, callback)
    };

    // 二叉树-中序遍历（从小到大顺序遍历出来）
    function midOrderTravelNode(root, callback) {
        if (root !== null) {
            midOrderTravelNode(root.left, callback);
            callback(root.key);
            midOrderTravelNode(root.right, callback);
        }
    }

    this.midOrderTravel = function (callback) {
        midOrderTravelNode(root, callback)
    };

    // 二叉树-后续序遍历（系统文件遍历）
    function lastOrderTravelNode(root, callback) {
        if (root !== null) {
            lastOrderTravelNode(root.left, callback);
            lastOrderTravelNode(root.right, callback);
            callback(root.key);
        }
    }

    this.lastOrderTravel = function (callback) {
        lastOrderTravelNode(root, callback)
    };

    this.show = function () {
        console.log('root==>', root)
    }
}

let BinaryTreeArr = [6, 3, 2, 4, 5];
let BinaryTreeObj = new BinaryTree();
BinaryTreeArr.forEach(item => {
    BinaryTreeObj.insert(item)
});
BinaryTreeObj.show();
BinaryTreeObj.preOrderTravel(function (key) {
    console.log('preOrderTravel==>', key)
});
BinaryTreeObj.midOrderTravel(function (key) {
    console.log('midOrderTravel==>', key)
});
BinaryTreeObj.lastOrderTravel(function (key) {
    console.log('lastOrderTravel==>', key)
});
```
### <a name="depthTraver">深度优先遍历</a>
```javascript
const DepthTraver = (node, nodeList = []) => {
    if (node != null) {
        nodeList.push(node);
        let children = node.children;
        for (let i = 0; i < children.length; i++) {
            DepthTraver(children[i], nodeList)
        }
    }
    return nodeList
};
let DepthNodeList = DepthTraver(document.getElementById('box'));
console.log('DepthNodeList==>', DepthNodeList);
```
### <a name="dreadthTraver">广度优先遍历</a>
```javascript
const BreadthTraver = (node) => {
    let nodeList = [];
    let stack = [];
    if(node != null) {
        stack.push(node);
        while (stack.length) {
            let item = stack.shift();
            let children = item.children;
            nodeList.push(item);
            for(let i=0; i<children.length; i++) {
                stack.push(children[i])
            }
        }
    }
    return nodeList;
};
let BreadthNodeList = BreadthTraver(document.getElementById('box'));
console.log('BreadthNodeList==>', BreadthNodeList);
```
### <a name="maxSum">求数组最大连续子集和</a>
```javascript
const maxSum = (arr=[]) => {
    let max = 0;
    let maxStartIndex = 0;
    let maxEndIndex = 0;
    for(let i=0; i<arr.length; i++) {
        let sum = 0;
        for(let j=i; j<arr.length; j++) {
            sum+=arr[j];
            if(sum>max) {
                max=sum;
                maxStartIndex = i;
                maxEndIndex = j
            }
        }
    }
    return {
        max: max,
        maxArr: arr.slice(maxStartIndex, maxEndIndex+1)
    }
};
let maxSumResult = maxSum([0,1,-3,4,7]);
console.log('maxSumResult==>', maxSumResult);
```
### <a name="arrToTree">把原始 list 转换成树形结构，parentId 为多少就挂载在该 id 的属性 children 数组下</a>
```javascript
let list =[
    {id:3,name:'部门C',parentId:1},
    {id:5,name:'部门E',parentId:2},
    {id:1,name:'部门A',parentId:0},
    {id:2,name:'部门B',parentId:0},
    {id:8,name:'部门H',parentId:4},
    {id:7,name:'部门G',parentId:2},
    {id:4,name:'部门D',parentId:1},
    {id:6,name:'部门F',parentId:3}
];
function convert(list) {
    const res = [];

    const map = list.reduce((pre, cur) => {
        pre[cur.id] = cur;
        return pre
    }, {});

    for (const item of list) {
        if (item.parentId === 0) {
            res.push(item);
            continue
        }
        if (item.parentId in map) {
            const parent = map[item.parentId];
            parent.children = parent.children || [];
            parent.children.push(item)
        }
    }
    console.log('list==>', list);
    console.log('map==>', map);
    return res
}

let newList = convert(list);
console.log('newList==>', newList);
```
### <a name="deepCopy">深度拷贝</a>
```javascript
function deepCopy (obj, cache = []) {
    // just return if obj is immutable value
    if (obj === null || typeof obj !== 'object') {
        return obj
    }

    // if obj is hit, it is in circular structure
    const hit = cache.find(c => c.original === obj);
    if (hit) {
        return hit.copy
    }

    const copy = Array.isArray(obj) ? [] : {};
    // put the copy into cache at first
    // because we want to refer it in recursive deepCopy
    cache.push({
        original: obj,
        copy
    });

    Object.keys(obj).forEach(key => {
        copy[key] = deepCopy(obj[key], cache)
    });

    return copy
}

var copyObj = deepCopy({
    a: 1,
    b: function() {
      
    },
    c: {name: 1},
    d: null,
    e: [1,2,3],
    f: true,
    g: 'abc',
    h: undefined
});
console.log('copyObj==>', copyObj);
```
