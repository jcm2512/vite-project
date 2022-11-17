<script>
  // @ts-nocheck

  import { onMount, afterUpdate } from "svelte";
  import { gsap } from "gsap";
  import { Draggable } from "gsap/Draggable";
  import * as Hammer from "hammerjs";

  gsap.registerPlugin(Draggable);

  let root = document.documentElement;

  // let CONTROLLER;

  // let DRAG_FRAME;
  // let TARGET;

  // let DOM_OBJECTS = [];
  // let OBJECTS = [
  //   { id: "1", img: "./candy_monster.png", pos: ["20px, 20px, 0px"] },
  //   { id: "2", img: "./candy_monster.png", pos: ["30px, 60px, 0px"] },
  // ];

  // OBJECTS.forEach((obj) => {
  //   let pos1 = Math.random() * document.body.getBoundingClientRect().width;
  //   let pos2 = Math.random() * document.body.getBoundingClientRect().height;
  //   obj.pos = [`${pos1}px, ${pos2}px, 0px`];
  // });

  let CONTROLLER = {
    MAIN: "",
    PAN: "",
    PINCH: "",
  };

  var currentScale = 1;
  var currentLeft = 0;
  var currentTop = 0;

  onMount(() => {
    // --------------------------------------
    // PINCH ZOOM & PAN WITH HAMMERJS
    // MODIFIED FROM A CODEPEN by David Linke
    // NEW ADDITIONS:
    //    ++ pan while pinching
    //    ++ keep pinch zoom centered around pointer

    var mc = new Hammer(CONTROLLER.MAIN, {
      domEvents: true,
    });

    currentScale = 1;
    currentLeft = 0;
    currentTop = 0;

    // zoom
    mc.get("pinch").set({ enable: true });
    mc.on("pinchstart", function (ev) {
      // on pinch zoom we eliminate the panning event listener
      // so that we dont have that weird movement after we end pinching
      mc.get("pan").set({ enable: false });

      // CONTROLLER.PAN.style.transform = `scale(${currentScale}) translate(${
      //   ev.center.x - currentLeft
      // }px, ${ev.center.y - currentTop}px)`;
    });
    mc.on("pinch", function (ev) {
      let s = currentScale * ev.scale;
      let x = currentLeft + ev.deltaX / currentScale;
      let y = currentTop + ev.deltaY / currentScale;
      CONTROLLER.PAN.style.transformOrigin = `${ev.center.x}px ${ev.center.y}px`;
      CONTROLLER.PAN.style.transform = `scale(${s}) translate(${x}px, ${y}px)`;
    });
    mc.on("pinchend", function (ev) {
      currentScale *= ev.scale;
      currentLeft = currentLeft + ev.deltaX / currentScale;
      currentTop = currentTop + ev.deltaY / currentScale;

      // once we have ended pinch zooming we fire off the panning event once again
      setTimeout(() => {
        mc.get("pan").set({ enable: true });
      }, 500);
    });

    mc.on("pan", function (ev) {
      let s = currentScale * ev.scale;
      let x = currentLeft + ev.deltaX / currentScale;
      let y = currentTop + ev.deltaY / currentScale;
      // CONTROLLER.PAN.style.transformOrigin = `${ev.center.x}px ${ev.center.y}px`;
      CONTROLLER.PAN.style.transform = `scale(${s}) translate(${x}px, ${y}px)`;
      console.log(ev.deltaX);
    });

    mc.on("panend", function (ev) {
      currentLeft = currentLeft + ev.deltaX / currentScale;
      currentTop = currentTop + ev.deltaY / currentScale;
    });
  });
</script>

<main id="main">
  <span id="preload-css" class="hit" />
  <div id="controller" bind:this={CONTROLLER.MAIN} />
  <div
    id="p-center"
    bind:this={CONTROLLER.PINCH}
    style="transform: scale({currentScale}) translate({currentLeft}px, {currentTop}px)"
  />
  <div id="d-center" bind:this={CONTROLLER.PAN} />

  <!-- <div class="outer">
    <div
      class="inner drag"
      bind:this={DRAG_FRAME}
      style="transform: scale({scene_scale})"
    > -->
  <!-- <span class="random_location">
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
      </span> -->
  <!-- </div>
  </div> -->
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

  #p-center {
    width: 1vw;
    height: 1vw;
    background-color: red;
    grid-column: 1;
    grid-row: 1;
    z-index: 100000;
    pointer-events: none;
    position: fixed;
  }

  #d-center {
    margin: -10vw -10vw;
    width: 20vw;
    height: 20vw;
    background-color: rgb(0, 255, 115);
    grid-column: 1;
    grid-row: 1;
    z-index: 100000;
    pointer-events: none;
    position: fixed;
    border-radius: 50%;
    opacity: 0.5;
    transition: transform 50ms ease;
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
