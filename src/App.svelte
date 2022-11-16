<script>
  // @ts-nocheck

  import { onMount, afterUpdate } from "svelte";
  import { gsap } from "gsap";
  import { Draggable } from "gsap/Draggable";
  import * as Hammer from "hammerjs";

  gsap.registerPlugin(Draggable);

  let DRAG_FRAME;

  let DOM_OBJECTS = [];
  let OBJECTS = [
    { id: "1", img: "./candy_monster.png", pos: ["20px, 20px, 0px"] },
    { id: "2", img: "./candy_monster.png", pos: ["30px, 60px, 0px"] },
  ];

  OBJECTS.forEach((obj) => {
    let pos1 = Math.random() * document.body.getBoundingClientRect().width;
    let pos2 = Math.random() * document.body.getBoundingClientRect().height;
    obj.pos = [`${pos1}px, ${pos2}px, 0px`];
  });

  onMount(() => {
    Draggable.create(DRAG_FRAME, {
      onDrag: function () {
        hitTest();
      },
    });
    let hammer = new Hammer(DRAG_FRAME, { domEvents: true });
    let el = DRAG_FRAME;
    let currentScale = 1;
    let currentLeft = 0;
    let currentTop = 0;
    let origin = { x: 0, y: 0 };

    // window.setTimeout(hammerPan, 50);

    // ZOOM
    hammer.get("pinch").set({ enable: true });
    hammer.on("pinchstart", function (event) {
      // disable panning
      // hammer.off("pan");
      origin = {
        x: event.changedPointers[0].clientX,
        y: event.changedPointers[0].clientY,
      };
      el.style.transformOrigin = `${origin.x}px ${origin.y}px`;
    });
    hammer.on("pinch", function (event) {
      el.style.transform = `scale(${
        currentScale * event.scale
      }) translate(${currentLeft}px, ${currentTop}px)`;
    });
    hammer.on("pinchend", function (event) {
      currentScale = currentScale * event.scale;
      currentLeft = currentLeft + event.deltaX / currentScale;
      currentTop = currentTop + event.deltaY / currentScale;
      // re-enable panning
      // window.setTimeout(hammerPan, 50);
    });

    // PAN
    // function hammerPan() {
    //   hammer.on("pan", function (event) {
    //     el.style.transform = `
    //     scale(${currentScale})
    //     translate(${currentLeft + event.deltaX / currentScale}px,
    //               ${currentTop + event.deltaY / currentScale}px)`;
    //   });
    // }

    // hammerPan();
    // hammer.on("panend", function (event) {
    //   currentLeft = currentLeft + event.deltaX / currentScale;
    //   currentTop = currentTop + event.deltaY / currentScale;
    // });
  });

  let target = {
    x: document.body.getBoundingClientRect().width / 2,
    y: document.body.getBoundingClientRect().height / 2,
  };

  let targetSize = 0.2;
  let prevHit;

  function hitTest() {
    let hit = document.elementFromPoint(target.x, target.y);
    if (prevHit && prevHit.id != hit.id) {
      prevHit.classList.remove("hit");
    }
    if (hit.classList.contains("obj")) {
      prevHit = hit;
      let hitObj = hit.getBoundingClientRect();
      let x = target.x - (hitObj.x + hitObj.width / 2);
      let y = target.y - (hitObj.y + hitObj.height / 2);

      if (
        Math.abs(x) < hitObj.width * targetSize &&
        Math.abs(y) < hitObj.height * targetSize
      ) {
        hit.classList.add("hit");
      }
    }
  }
</script>

<main id="main">
  <div id="center" />
  <span id="preload-css" class="hit" />
  <div class="outer">
    <div class="inner drag" bind:this={DRAG_FRAME}>
      <span class="random_location">
        {#each OBJECTS as obj, index}
          <img
            id="object_{index}"
            class="obj"
            data-hitTime="0"
            bind:this={DOM_OBJECTS[index]}
            src={obj.img}
            alt="dragobject"
            style="transform: translate3d({obj.pos});"
          />
        {/each}
      </span>
    </div>
  </div>
</main>

<style>
  #main {
    display: grid;
  }
  #center {
    width: 5vw;
    height: 5vw;
    background-color: red;
    grid-column: 1;
    grid-row: 1;
    z-index: 100000;
    pointer-events: none;
    position: fixed;
    top: 50%;
    left: 50%;
    -webkit-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
  }
  .outer {
    display: flex;
    align-items: center;
    justify-content: center;
    /* overflow: hidden; */
    height: 100vh;
    width: 100%;
    background-color: grey;
    grid-column: 1;
    grid-row: 1;
  }
  .obj {
    position: absolute;
    width: 30vw;
  }

  .drag {
    z-index: 0;
    width: 400vw;
    height: 100vh;
    background-color: darkgrey;
  }

  .hit {
    background-color: green;
  }
</style>
