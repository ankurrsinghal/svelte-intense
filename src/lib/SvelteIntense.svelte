<script lang="ts">
  import { onMount } from "svelte";

  const KEYCODE_ESC = 27;
  const MIN_DIST_THRESHOLD = 0.0001;
  const DEFAULT_MOVE_SPEED = 0.02;

  export let src: string;
  export let thumbnailSrc: string;
  export let className: string;
  export let caption: string;
  export let title: string;
  export let moveSpeed: number = DEFAULT_MOVE_SPEED;
  export let onClick: (e: MouseEvent) => void = () => {};
  export let invert: boolean = false;
  export let vertical: boolean = false;

  let distance = 0;
  let loaded = false;
  let mouseDest = {
    x: 0,
    y: 0,
  };
  let mouseCurr;
  let transform = "";
  let triggered = false;
  let intervalKey;
  let imgRef;
  let internalWindow;
  let initialOverflowValue;

  onMount(() => {
    internalWindow = window;
    mouseCurr = {
      x: window.innerWidth / 2,
      y: window.innerHeight / 2,
    };
    initialOverflowValue = document.body.style.overflow || "unset";
  });

  function positionTarget() {
    if (!imgRef) {
      return;
    }

    const newX = mouseCurr.x + (mouseDest.x - mouseCurr.x) * moveSpeed;
    const newY = mouseCurr.y + (mouseDest.y - mouseCurr.y) * moveSpeed;
    const dist = Math.abs(newX - mouseCurr.x) + Math.abs(newY - mouseCurr.y);

    if (dist > MIN_DIST_THRESHOLD) {
      const newMouseCurr = {
        x: newX,
        y: newY,
      };

      let newTransform = "";
      let newDistance = -1;
      if (vertical) {
        if (newY !== distance) {
          const overflowPosition = calcPosition(newY, window.innerHeight);
          const yOverflow = window.innerHeight - imgRef?.offsetHeight;
          const position = yOverflow * overflowPosition;
          newTransform = `translate3d(0px, ${position}px, 0px)`;
          newDistance = newY;
        }
      } else {
        if (newX !== distance) {
          const overflowPosition = calcPosition(newX, window.innerWidth);
          const xOverflow = window.innerWidth - imgRef?.offsetWidth;
          const position = xOverflow * overflowPosition;
          newTransform = `translate3d(${position}px, 0px, 0px)`;
          newDistance = newX;
        }
      }

      transform = newTransform;
      distance = newDistance;

      mouseCurr = newMouseCurr;
    }
  }

  function calcPosition(current: number, total: number) {
    return invert ? (total - current) / total : current / total;
  }

  function addEventListeners() {
    imgRef.addEventListener("mousemove", (e) =>
      _onMouseMove(e.clientX, e.clientY)
    );
    imgRef.addEventListener("touchmove", (e) => _onTouchMove(e.touches[0]));
    imgRef.addEventListener("click", hideViewer);
    window.addEventListener("keyup", _onKeyUp);
    loop();
  }

  function removeEventListeners() {
    imgRef.removeEventListener("mousemove", (e) =>
      _onMouseMove(e.clientX, e.clientY)
    );
    imgRef.removeEventListener("touchmove", (e) => _onTouchMove(e.touches[0]));
    imgRef.removeEventListener("click", hideViewer);
    window.removeEventListener("keyup", _onKeyUp);
  }

  // Lock scroll on the document body.
  function lockBody() {
    document.body.style.overflow = "hidden";
  }

  // Unlock scroll on the document body.
  function unlockBody() {
    document.body.style.overflow = initialOverflowValue;
  }

  // Stop animation
  function stop() {
    if (intervalKey) {
      window.cancelAnimationFrame(intervalKey);
    }
    intervalKey = undefined;
    removeEventListeners();
    unlockBody();
  }

  // Start animation
  function loop() {
    const newIntervalKey = window.requestAnimationFrame(loop);
    intervalKey = newIntervalKey;
    positionTarget();
  }

  // Events
  function maximize(e: MouseEvent) {
    e.preventDefault();

    if (onClick) {
      onClick(e);
    }

    mouseDest = {
      x: e.clientX,
      y: e.clientY,
    };

    triggered = true;
    lockBody();
  }

  function _onKeyUp(e: KeyboardEvent) {
    e.preventDefault();

    if (e.keyCode === KEYCODE_ESC) {
      hideViewer();
    }
  }

  function onLoad() {
    loaded = true;
    addEventListeners();
  }

  function _onMouseMove(x: number, y: number) {
    mouseDest = {
      x,
      y,
    };
  }

  function _onTouchMove(touch: Touch) {
    mouseDest = {
      x: touch.clientX,
      y: touch.clientY,
    };
  }

  // View helpers
  function hideViewer() {
    triggered = false;
    stop();
  }

  $: height =
    internalWindow && (vertical ? "" : `${internalWindow.innerHeight}px`);
  $: width =
    internalWindow && (vertical ? `${internalWindow.innerWidth}px` : "");
  $: transformStyle = transform;
</script>

<div class="ri-wrapper">
  <div
    class={`${className} ri-trigger ${triggered ? " ri-clicked" : ""}`}
    style:background-image="url({thumbnailSrc})"
    on:click={maximize}
    aria-hidden
  >
    {#if triggered}
      <div class="ri-loader"></div>
    {/if}
  </div>
  {#if triggered}
    <figure class="ri-container" style:opacity={loaded ? 1 : 0}>
      <img
        bind:this={imgRef}
        {src}
        style:height
        style:width
        style:transform={transformStyle}
        on:load={onLoad}
        alt={caption}
      />
      <figcaption class="ri-caption-container">
        <h1 class="ri-title">{title}</h1>
        <h2 class="ri-caption">{caption}</h2>
      </figcaption>
    </figure>
  {/if}
</div>

<style>
  .ri-wrapper,
  .ri-cursor-plus {
    cursor:
      url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyRpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoTWFjaW50b3NoKSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo4M0UzRkZGRUY4ODQxMUUzQjg5REQwNUQyQTFENTVGOSIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDo4M0UzRkZGRkY4ODQxMUUzQjg5REQwNUQyQTFENTVGOSI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjUyNjc4QUY1Rjg0QjExRTNCODlERDA1RDJBMUQ1NUY5IiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOjUyNjc4QUY2Rjg0QjExRTNCODlERDA1RDJBMUQ1NUY5Ii8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+x6eaoAAABeVJREFUeNrcWn1MW1UUv604GZgFzSzFyRifDR/SpWzIPy5QCIlGsiZGJ05JDEELiYBEp1FKgoLRSaDwB64mmEyjxJlFDCbGBRiZf4zwlcEoDd8DdBRcpDGAWFnx/OAWH6x09Gttd5LDve/x3ru/X8+555173xExN8n6+no4NRmkx0hlpEdID5IGk4pIl0hvkd4gHSHtJb0sEolmXRzXLeAPk2pI9evOi54/I8JZIlCRkzfLqfmA9HlSMc6tra0tLywsDE5MTAzo9fqpwcHB+f7+ftPU1NTf+H9kZOR+hUIRkpycHJqYmBgZHR0tl0gkyQEBAcH8sRbSi6TVZKUBRy0icpCAhJqzpHm412Kx/Ds9PX2lo6OjvaSkpGd5edniyPOCg4PF9fX1x5VKZWZERMQJsVj8IIYh/Yr0DBFacDsRuuEFar4gDSEC/xgMhp/Ky8svtLS03HLHHFOpVAerqqpejI+Pf44IPUSnTKSvE5nv3UKELtxHjZa0EMdGo7G7rKxM29zcPM88ILm5uaG1tbUlUqn0KX7qHGkJETI7TYQuepj7bTbcqLOz81xmZmYLuwfS3t6uSk9PV3N3u4T5SGSWdiMitkMiiD8g22w2L1ZXV5feKxIQjIUxMTYwAAvHZFNEdtzpZ1LlysqKsbi4+ExTU9PvzAuSn59/qKGh4WxQUJAUhiJ9Vuhmd7MI5oRydXX1j6Kiore9RQKCsYEBWGAo0npb14ltWOMUJjbNibWamprK8+fPzzEvCzAACzDRoZowvmTXteiCx6nRI8TSZGvIysr6kfmQtLW1naS5U8xDcyK52E2bUYtOfkPNy/Pz870U/t5lPigU/j8NDQ1FPvctETl9xxyhEwqEcTKfWaPRNDAfFWADRmDlmLdbhE4itJ6kPOliUlJSI/NhGRoaKqJ8DXkeXF+1ZRGeeebgpYe0w4sYH9jR2hRg5BM/B/mo0LVeQX92dvZXd+VOnhRgJKxXOP7XhERO488lEncPStZuh7r7uQKspzbmCF/ZzWA9ERISonI0Fd8LkY2BRKLMPbrWbUFrdwlgMpl+oPUM8sFoWESJf9CiaMDdJDwpwEqYr/PDDDFfY7PJycnrzM+EMA/y7lEQiUNvdHR0xt+IjI+PWzHLQCQave7u7t/8jUhXV5cVcxSIPMZP/ulvRIaHh//i3RBErXUHoordyOSsCMbec9SCSCSSAMoLf6GuWczuEwkghXkOyOXyoIGBgRUXf1FX3yMOiUwmsy59l2ARrLxYWlrao/5mhYSEhAO8uwgiE+ilpqY+4W9E6Me3Yp4EkVH04uLiDvsbkZiYGCvmERDp3QjEUVFP+hsRAeZrINLBQ1kyEjF/IQGswMwPL4v59wkDssi6uroUfyECrDzzHbPOEQg2HVg2ibsHRNj1ROgVYP1OuLACEUt4ePjT2BX3dWsAI2E9wTa/qXy5RYR+sRvUtGLDGFv7XsR4e0drU4CRsOJl3ko6tW07iOQjvIjj4+NzCgoKDvmqNYANGNnmB6EP79gO4unExgad0WjsCQsLe88XiczNzX0ilUqPs9026Li8Q2rChdie9DUSwMRJmDjWLRHviDA3Gf8ylZGRUajRaOJ8hURFRYUMmPhhIcfKbLqWwMU+p0aNrXy1Wv2Wt3fk8/LypDqdThsYGIhFoI5IqAVY7RLx1Q89yEKe2fOHHn4hIsNVPKCxsbG+srJS5g13wticxFVg2u2j6P3/MZRbZolbRocHKpXKNxH+YG5PuhLGwFichI5bYskuVgc2GIQFA2aDwdDqoYKBHCKwj4fYN4jAhbvgcowIvwklHJ+RvsoEJRzkBm2lpaW9zpRwaLXaY+SuWTtKOL7Ge8IjJRw7bj5Kzftse1HNEopqsPU6MjIy3d/fP9fX17c4Nja2UVQTGxu7PyUl5RGFQhEmk8kiaFGUROsJOU/FGfu/qOZjInDNASyuu4OgzGnYhTIng9fKnHYjxTYLz5BCICM4wjYLz6y/uLDwDPsEPWyz8GzGxXE32v8EGADbuW2sOaxEjQAAAABJRU5ErkJggg==")
        25 25,
      no-drop;
  }

  .ri-container,
  .ri-cursor-x {
    cursor:
      url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyRpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoTWFjaW50b3NoKSIgeG1wTU06SW5zdGFuY2VJRD0ieG1wLmlpZDo3Q0IyNDI3M0FFMkYxMUUzOEQzQUQ5NTMxMDAwQjJGRCIgeG1wTU06RG9jdW1lbnRJRD0ieG1wLmRpZDo3Q0IyNDI3NEFFMkYxMUUzOEQzQUQ5NTMxMDAwQjJGRCI+IDx4bXBNTTpEZXJpdmVkRnJvbSBzdFJlZjppbnN0YW5jZUlEPSJ4bXAuaWlkOjdDQjI0MjcxQUUyRjExRTM4RDNBRDk1MzEwMDBCMkZEIiBzdFJlZjpkb2N1bWVudElEPSJ4bXAuZGlkOjdDQjI0MjcyQUUyRjExRTM4RDNBRDk1MzEwMDBCMkZEIi8+IDwvcmRmOkRlc2NyaXB0aW9uPiA8L3JkZjpSREY+IDwveDp4bXBtZXRhPiA8P3hwYWNrZXQgZW5kPSJyIj8+soZ1WgAABp5JREFUeNrcWn9M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")
        25 25,
      no-drop;
  }

  .ri-wrapper .ri-clicked {
    box-shadow: inset 0 0 0 500px rgba(0, 0, 0, 0.2);
  }

  .ri-container {
    background-color: rgba(0, 0, 0, 0.8);
    width: 100%;
    height: 100%;
    position: fixed;
    top: 0px;
    left: 0px;
    overflow: hidden;
    z-index: 999999;
    margin: 0px;
    transition: opacity 150ms cubic-bezier(0, 0, 0.26, 1);
  }

  .ri-caption-container {
    font-family: Georgia, Times, "Times New Roman", serif;
    position: fixed;
    bottom: 0px;
    left: 0px;
    padding: 20px;
    color: #fff;
    word-spacing: 0.2px;
    text-shadow: -1px 0px 1px rgba(0, 0, 0, 0.4);
  }

  .ri-title {
    margin: 0px;
    padding: 0px;
    font-weight: normal;
    font-size: 40px;
    letter-spacing: 0.5px;
    line-height: 35px;
    text-align: left;
  }

  .ri-caption {
    margin: 0px;
    padding: 0px;
    font-weight: normal;
    font-size: 20px;
    letter-spacing: 0.1px;
    max-width: 500px;
    text-align: left;
    background: none;
    margin-top: 5px;
  }

  .ri-trigger {
    width: 200px;
    height: 200px;
    background-size: cover;
    background-position: 50% 50%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .ri-loader {
    display: inline-block;
    width: 50px;
    height: 50px;
    border: 3px solid rgba(255, 255, 255, 0.2);
    border-radius: 50%;
    border-top-color: #fff;
    animation: spin 1s ease-in-out infinite;
    -webkit-animation: spin 1s ease-in-out infinite;
  }
  @keyframes spin {
    to {
      -webkit-transform: rotate(360deg);
    }
  }
  @-webkit-keyframes spin {
    to {
      -webkit-transform: rotate(360deg);
    }
  }
</style>
