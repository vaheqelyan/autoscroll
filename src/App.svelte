<script>
export let name;

import { onMount, tick } from "svelte";

Array.prototype.move = function (from, to) {
  this.splice(to, 0, this.splice(from, 1)[0]);
  return this;
};

export const debounce = (fn, ms = 0) => {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), ms);
  };
};

import Move from "./Move.svelte";

/*const boundMap = {};
onMount(() => {
    const z = contaon

})*/

const id = () => "_" + Math.random().toString(36).substr(2, 9);

const randomNumber = (min, max) => Math.random() * (max - min) + min

let data = new Array(10).fill().map((_, index) => ({ 
    value: index, id: id(), height: randomNumber(70, 140)
}));

let start = false;
let activeIndex = -1;
let initialIndex = 0;
let _height = 0;

const onStart = (e) => {
  start = true;
  activeIndex = e.detail.index;
  initialIndex = e.detail.index;
  _height = e.detail.height;
};

const update = (e) => {
  if (start) {
      let newIndex = Math.min(Math.max(0, e.detail.newIndex), data.length - 1);
      let copy = data.slice();

    data = copy.move(activeIndex, newIndex);

    activeIndex = newIndex;
  }
};

const end = () => {
  activeIndex = false;
  start = false;
  initialIndex = 0;
};

//const handleRepaint = debounce(update, 100);
const handleRepaint = update;

let container = null;


const boundMap = {};
onMount(() => {
    const z = container.firstChild.children;

for (var i = 0; i < z.length; i++) {
        boundMap[i] = z[i].getBoundingClientRect();
}
console.log(boundMap);


})
</script>

<div class=scroll bind:this={container}>
    <div class=inner style="height: 1000px;">
{#each data as item, index (item.id)}
    <Move value={item} {start} {activeIndex} {container} on:update={handleRepaint} index={index} on:start={onStart} initIndex={initialIndex} on:end={end} 
          activeHeight={_height}
          {boundMap}
          />
{/each}
    </div>
</div>

<style>
.scroll {
  overflow-y: scroll;
  max-height: 300px;
  max-width: 300px;
  width: 100%;
  position: relative;
  left: 300px;
  top: 100px;
}
</style>
