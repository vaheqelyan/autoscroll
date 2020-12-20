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

const id = () => "_" + Math.random().toString(36).substr(2, 9);

let data = new Array(10).fill().map((_, index) => ({ value: index, id: id() }));
let start = false;
let activeIndex = -1;
let initialIndex = 0;

const onStart = (e) => {
  start = true;
  activeIndex = e.detail.index;
  initialIndex = e.detail.index;
};

const update = (e) => {
  if (start) {
    let newIndex = e.detail.newIndex;
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

const handleRepaint = debounce(update, 100);

let container = null;
</script>

<div class=scroll bind:this={container}>
    <div class=inner style="height: 1000px;">
{#each data as item, index (item.id)}
    <Move value={item} {start} {activeIndex} {container} on:update={handleRepaint} index={index} on:start={onStart} initIndex={initialIndex} on:end={end} />
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
