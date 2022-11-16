<script>
  // @ts-nocheck

  import { onMount, afterUpdate } from "svelte";
  import { gsap } from "gsap";
  import { Draggable } from "gsap/Draggable";
  import * as Hammer from "hammerjs";

  gsap.registerPlugin(Draggable);

  let DRAG_FRAME;
  let TARGET;

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

  const object_scale = 0.2;
  let scene_scale = 5.0;

  onMount(() => {
    // Draggable.create(DRAG_FRAME, {
    //   onDrag: function () {
    //     hitTest();
    //   },
    // });

    const target = TARGET;

    var el = DRAG_FRAME;
    var mc = new Hammer(el, {
      domEvents: true,
    });

    const max_scale = 5.0;
    const min_scale = 1.0;

    var currentScale = 5.0;
    var currentLeft = 0;
    var currentTop = 0;

    // zoom
    mc.get("pinch").set({ enable: true });
    mc.on("pinchstart", function (ev) {
      // on pinch zoom we eliminate the panning event listener
      //so that we dont have that weird movement after we end pinching
      mc.off("pan");
    });
    mc.on("pinch", function (ev) {
      el.style.transform =
        "scale(" +
        limitScale(currentScale * ev.scale) +
        ") translate(" +
        currentLeft +
        "px," +
        currentTop +
        "px)";
    });
    mc.on("pinchmove", function (ev) {
      hitTest(target);
    });
    mc.on("panmove", function (ev) {
      hitTest(target);
      if (ev.target.getBoundingClientRect().x > 0);
    });
    mc.on("pinchend", function (ev) {
      currentScale = limitScale(currentScale * ev.scale);

      // once we have ended pinch zooming we fire off the panning event once again
      window.setTimeout(hammerPan, 50);
    });

    function limitScale(scale) {
      if (scale >= max_scale) {
        return max_scale;
      }
      if (scale <= min_scale) {
        return min_scale;
      }
      return scale;
    }

    function limitPosX(posx) {
      if (posx > 0) {
        return 0;
      }
      return posx;
    }

    // panning function
    function hammerPan() {
      mc.on("pan", function (ev) {
        el.style.transform =
          "scale(" +
          limitScale(currentScale) +
          ") translate(" +
          (currentLeft + ev.deltaX) / limitScale(currentScale) +
          "px," +
          (currentTop + ev.deltaY / limitScale(currentScale)) +
          "px)";
      });
    }

    hammerPan();
    mc.on("panend", function (ev) {
      currentLeft = currentLeft + ev.deltaX / limitScale(currentScale);
      currentTop = currentTop + ev.deltaY / limitScale(currentScale);
    });
  });

  let targetSize = 0.2;
  let prevHit;

  function hitTest(target) {
    let targetObj = target.getBoundingClientRect();
    let targetpos = {
      x: targetObj.x + targetObj.width / 2,
      y: targetObj.y + targetObj.height / 2,
    };
    let hit = document.elementFromPoint(targetpos.x, targetpos.y);
    if (prevHit && prevHit.id != hit.id) {
      prevHit.classList.remove("hit");
    }
    if (hit.classList.contains("obj")) {
      prevHit = hit;

      //// SIMPLE HIT TEST
      hit.classList.add("hit");

      //// VARIABLE SIZE HIT TEST
      // let hitObj = hit.getBoundingClientRect();
      // let dist_to_targetObj_x = targetpos.x - (hitObj.x + hitObj.width / 2);
      // let dist_to_targetObj_y = targetpos.y - (hitObj.y + hitObj.height / 2);
      // if (
      //   Math.abs(dist_to_targetObj_x) < hitObj.width &&
      //   Math.abs(dist_to_targetObj_y) < hitObj.height
      // ) {
      //   hit.classList.add("hit");
      // }
    }
  }
</script>

<main id="main">
  <div id="controller" />
  <div id="center" bind:this={TARGET} />
  <span id="preload-css" class="hit" />
  <div class="outer">
    <div
      class="inner drag"
      bind:this={DRAG_FRAME}
      style="transform: scale({scene_scale})"
    >
      <span class="random_location">
        {#each OBJECTS as obj, index}
          <img
            id="object_{index}"
            class="obj"
            data-hitTime="0"
            bind:this={DOM_OBJECTS[index]}
            src={obj.img}
            alt="dragobject"
            style="transform: translate3d({obj.pos}); scale: {object_scale}"
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
  #controller {
    height: 100vh;
    width: 100%;
    grid-column: 1;
    grid-row: 1;
    z-index: 999999;
  }

  #center {
    width: 1vw;
    height: 1vw;
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
    width: 100vw;
    height: 100vh;
    background-color: darkgrey;
    transition: transform 80ms ease;
  }

  .hit {
    background-color: green;
  }
</style>
