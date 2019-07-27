冒泡排序
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
选择排序
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
插入排序
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
合并排序
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
快速排序
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
