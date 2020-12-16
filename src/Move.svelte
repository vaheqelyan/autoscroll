<script>
export let value;
export let index;
export let start;
export let activeIndex;
export let container;
export let initIndex;

import { createEventDispatcher, beforeUpdate } from "svelte";

import { tick } from "svelte";

const dispatch = createEventDispatcher();

let xyRef = { x: 0, y: 0 };
let newXY = { x: 0, y: 0 };
let initXY = { x: 0, y: 0 };
let active = false;
let intervalId = false;

let sign = 0;
let _top = 0;
let _containerTop = 0;
let _containerBottom = 0;

let debounce = true;

const fn = () => {
  console.log("move");
};

const pointerdown = ({ clientX, clientY, target }) => {
  initXY = {
    x: clientX,
    y: clientY,
  };

  dispatch("start", {
    index,
  });

  _top = target.getBoundingClientRect().y;
  _containerTop = container.getBoundingClientRect().y;
  _containerBottom = container.getBoundingClientRect().bottom;
  active = true;
  window.addEventListener("pointermove", pointermove);

  window.addEventListener("pointerup", pointerup);
};

const pointerup = ({ clientX, client }) => {
  window.removeEventListener("pointermove", pointermove);
  window.removeEventListener("pointerup", pointerup);
  xyRef = { x: 0, y: 0 };
  newXY = { x: 0, y: 0 };
  initXY = { x: 0, y: 0 };

  clearInterval(intervalId)
  intervalId = false

  active = false;
  dispatch("end", {});
};

let _t = 0;

const pointermove = ({ clientX, clientY }) => {
  newXY = {
    x: initXY.x - clientX,
    y: initXY.y - clientY,
  };

  let y = xyRef.y - newXY.y;

  //let xin = Math.round(((initIndex * 100) + y) / 100);

  let topSensor = _top + y < _containerTop + 20;
  let bottomSensor = _top + y + 100 > _containerBottom - 20;

    //console.log( (_top + y + 100 - (_containerBottom - 20))  / 20);
    //console.log( (_containerTop + 20 - (_top+ y) ) / 20);

    _t = ( (_top + y + 100) - (_containerBottom - 20) ) / 20;

    sign = topSensor ? -1 : bottomSensor ? 1 : 0;

    console.log(topSensor,bottomSensor, sign)

  if (topSensor || bottomSensor) {

    if (!intervalId) {
      intervalId = setInterval(() => {
          container.scrollTop += 2 * sign;
      }, 10);
    }

  } else {
    console.log('remove event')
    clearInterval(intervalId);
    intervalId = false;
    sign = 0;
  }

  /*dispatch("update", {
    newIndex: xin,
  });*/
};
</script>


<div class:fixed={active} class=item on:pointerdown={pointerdown} style="
{active ? `transform: translate(${xyRef.x - newXY.x}px, ${xyRef.y - newXY.y}px); 

top: ${_top}px;
width: 285px;
` : ''}


{(start && activeIndex !== index ) ? `transform: translateY(${ index > activeIndex ? 100 : 0 }px)` : ''}


">
{value.value}
</div>

<style>
.item {
  height: 100px;
  width: 100%;
  box-shadow: inset 0px 0px 4px red;
  user-select: none;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
  position: static;
}

.transition {
  transition: transform 0.2s;
}

.fixed {
  position: fixed;
}
</style>

