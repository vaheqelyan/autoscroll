<script>
export let value;
export let index;
export let start;
export let activeIndex;
export let container;
export let initIndex;
export let activeHeight = 0;
export let boundMap;

import { createEventDispatcher, beforeUpdate } from "svelte";

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

let _height = 0;

const fn = () => {
  console.log("move");
};

const pointerdown = ({ clientX, clientY, target }) => {
  initXY = {
    x: clientX,
    y: clientY,
  };


  _height = target.getBoundingClientRect().height;

  dispatch("start", {
      index,
      height: _height,
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

  clearInterval(intervalId);
  intervalId = false;

  active = false;
  dispatch("end", {});
};

let _t = 0;
let vel = 1;



let oldY = 0;

const pointermove = ({ clientX, clientY }) => {
  newXY = {
    x: initXY.x - clientX,
    y: initXY.y - clientY,
  };

  let y = xyRef.y - newXY.y;

  // Design for detecting autoscroll
  let topSensor = _top + y < _containerTop + 20;
  let bottomSensor = _top + y + _height > _containerBottom - 20;

  // Designed for direction velocity
  let velocityBottom = Math.max(0, (_top + y + _height - (_containerBottom - 20)) / 20);
  let velocityTop = Math.max(0, (_containerTop + 20 - (_top + y)) / 20);

  // Designed for detecting direction of movement
  sign = topSensor ? -1 : bottomSensor ? 1 : 0;

  // Update velocity
  vel = sign === -1 ? velocityTop : velocityBottom;
  vel = vel * 2;


  // If autoscroll
  if (topSensor || bottomSensor) {
    // and if setInterval id is invalid
    if (!intervalId) {
      // init autoscroll frame
      intervalId = setInterval(() => {
        container.scrollTop += 2 * vel * sign;
      }, 10);
    }
  } else if(intervalId) {
    // Clear timer when auto scroll is not true
    clearInterval(intervalId);
    intervalId = false;
    sign = 0;
  }

        if (y < oldY) {
          console.log('top');
        } else if (y > oldY) {
          //console.log('bottom');
        }

    let xin = -1;


    //let s1 = _top + container.scrollTop + y + _height > boundMap[index + 1]?.bottom - boundMap[index+ 1]?.height / 2;
    //let s2 = _top + y + container.scrollTop < boundMap[index - 1]?.bottom - boundMap[index - 1]?.height / 2;

    //console.log(_top + container.scrollTop + y);

    let yy = _top + y + container.scrollTop;

    //console.log(boundMap[index - 1]?.y  - yy, _top - yy, boundMap[index + 1]?.y - yy);


    if(boundMap[index - 1]?.y  - yy > 0) {
dispatch("update", {
        newIndex: index - 1,
      });
        return
    }

    if(boundMap[index + 1]?.y - yy < 0) {

dispatch("update", {
        newIndex: index + 1,
      });
        return;
    }

    /*if(Math.abs(boundMap[index - 1]?.y  - yy) > Math.abs(boundMap[index + 1]?.y - yy)) {
        console.log(index - 1);
      dispatch("update", {
        newIndex: index - 1,
      });
    } else {
        console.log(index+1);
        dispatch("update", {
        newIndex: index + 1,
      });

    }*/

    /*if(s1) {
      dispatch("update", {
        newIndex: index + 1,
      });
    } else if(s2) {
      dispatch("update", {
        newIndex: index - 1,
      });
    }*/

    /*if(y > oldY) {
      if(_top + container.scrollTop + y + _height > boundMap[index + 1].bottom - boundMap[index+ 1].height / 2) {
        xin = index + 1;
      }
    } else if(y < oldY) {

        if(_top + y + container.scrollTop < boundMap[index - 1].bottom - boundMap[index - 1].height / 2) {
            xin = index - 1;
        }

    }*/

    if(xin !== -1) {
      dispatch("update", {
        newIndex: xin,
      });
    }


        oldY = y;

    //let xin = -1;
    /*if(sign === 1) {
      if(_top + container.scrollTop + y + _height > boundMap[index + 1].bottom - boundMap[index+ 1].height / 2) {
        xin = index + 1;
      }
    } else if(sign === -1) {

    }

    if(xin !== -1) {

dispatch("update", {
    newIndex: xin,
  });*/

    //}

/*dispatch("update", {
    newIndex: xin,
  });*/

    /*for(var i = index; i < 10;i++) {
      let f = boundMap[i];
        if(_top + container.scrollTop + y > f.top) {
            console.log(i);
        }
    }*/

  /*const xin = Math.round((_top + container.scrollTop + y) / 100) - 1;
*/
/*  dispatch("update", {
    newIndex: xin,
  });*/

};
</script>


<div class:fixed={active} class=item on:pointerdown={pointerdown} style="
{active ? `transform: translate(${xyRef.x - newXY.x}px, ${xyRef.y - newXY.y}px); 

top: ${_top}px;
width: 285px;
` : ''}


{(start && activeIndex !== index ) ? `transform: translateY(${ index > activeIndex ? activeHeight : 0 }px)` : ''};

height: {value.height}px

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

