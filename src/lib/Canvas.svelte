<script>
  import { onMount as om } from "svelte";
  // import { imgbbUploader } from "imgbb-uploader";
  // import { imgbbUploader } from "imgbb-uploader/lib/esm/index.js";
  import axios from "axios";

  export let width = 420;
  export let height = 420;
  let n = 15;
  export let color = "#0000ff";
  export let background = "#001";

  let canvas;
  let context;
  let isDrawing;
  let start;

  let t, l;

  om(() => {
    context = canvas.getContext("2d");
    context.lineWidth = 7;
    context.beginPath();
    context.rect(0, 0, 400, 400);
    context.fillStyle = background;
    context.fill();
    handleSize();
  });

  $: if (context) {
    context.strokeStyle = color;
  }

  const handleStart = ({ offsetX: x, offsetY: y }) => {
    if (color === background) {
      context.clearRect(0, 0, width, height);
    } else {
      isDrawing = true;
      start = { x, y };
    }
  };

  const handleEnd = () => {
    isDrawing = false;
  };

  const drawRect = (x, y, alpha, size) => {
    context.beginPath();
    // context.arc(x, y, size, 0, 2 * Math.PI);
    context.rect(x, y, size, size);
    context.fillStyle = color + alpha.toString(16);
    context.fill();
  };

  const drawDitheredRectangles = (x, y) => {
    /* draw rectangle in center, then get gradually less opaque */
    // drawRect(x, y, "0a", 28, 0);
    drawRect(x, y, 157, 20);
    drawRect(x - 10, y, Math.floor(36 * Math.random()), 16);
    drawRect(x, y - 10, Math.floor(51 * Math.random()), 16);
    drawRect(x, y + 10, Math.floor(40 * Math.random()), 16);
    drawRect(x + 10, y, Math.floor(36 * Math.random()), 16);
  };

  const handleMove = ({ offsetX: x1, offsetY: y1 }) => {
    if (!isDrawing) return;
    let r = Math.floor(y1 / n) / 2;
    let c = Math.floor(x1 / n) / 2;
    const { x, y } = start;
    // context.moveTo(c * 28, r * 28);
    // context.beginPath();
    // context.rect(c * 28, r * 28, 28, 28);
    // context.fillStyle = color;
    // context.fill();
    drawDitheredRectangles(c * 28, r * 28);

    start = { x: x1, y: y1 };
  };

  const handleSize = () => {
    const { top, left } = canvas.getBoundingClientRect();
    t = top;
    l = left;
  };

  let url = "";
  const uploadImage = (imgb64) => {
    // fileinput is the file/index 0 from input-type-file eg: e.target.myfileinput.files[0]
    const formData = new FormData();
    let key = import.meta.env.VITE_IMGBB_API_KEY;
    formData.append("image", imgb64.slice(22)); // has to be named 'image'!
    let apiresponse = axios
      .post("https://api.imgbb.com/1/upload?key=" + key, formData)
      .then((res) => {
        console.log(res);
        url = res.data.data.display_url;
        console.log(url);
      })
      .catch((error) => {
        return null;
      });

    return apiresponse;
  };

  const handleSave = (toUrl) => {
    const resized = document.createElement("canvas");
    resized.width = 28;
    resized.height = 28;
    let resized_ctx = resized.getContext("2d");
    resized_ctx.drawImage(canvas, 0, 0, 400, 400, 0, 0, 28, 28);
    const link = document.createElement("a");
    link.download = "image.png";
    link.href = resized.toDataURL();
    if (toUrl) {
      uploadImage(resized.toDataURL());
    } else {
      link.click();
    }
  };

  const handleClear = () => {
    context.clearRect(0, 0, canvas.width, canvas.height);
    url = "";
  };

  let urlButton;

  const copyURLToClipboard = () => {
    navigator.clipboard.writeText(urlButton.textContent);
  };
</script>

<svelte:window on:resize={handleSize} />
<div class="container">
  <div class="saves">
    {#if url.length != 0}
      <button
        class="url-display"
        on:click={copyURLToClipboard}
        bind:this={urlButton}
        >{url}
      </button>
    {/if}
    <button on:click={() => handleSave(true)}>Upload to URL</button>
    <button on:click={() => handleSave(false)}>Save Locally</button>
  </div>
  <button on:click={handleClear}>Clear</button>

  <canvas
    id="canvas"
    {width}
    {height}
    style:background
    bind:this={canvas}
    on:mousedown={handleStart}
    on:touchstart={(e) => {
      const { clientX, clientY } = e.touches[0];
      handleStart({
        offsetX: clientX - l,
        offsetY: clientY - t,
      });
    }}
    on:mouseup={handleEnd}
    on:touchend={handleEnd}
    on:mouseleave={handleEnd}
    on:mousemove={handleMove}
    on:touchmove={(e) => {
      const { clientX, clientY } = e.touches[0];
      handleMove({
        offsetX: clientX - l,
        offsetY: clientY - t,
      });
    }}
  />
</div>

<style>
  canvas {
    width: 420px;
    min-height: 420px;
  }

  .container {
    margin-top: 2px;
    display: flex;
    flex-direction: column;
    max-height: 90vh;
  }

  button {
    border: 1px solid aliceblue;
    margin: 1px;
    transition: all 0.2s;
  }

  button:active {
    rotate: -3deg;
    scale: 1.2;
    background-color: rgba(50, 48, 48, 0.211);
  }

  button:hover {
    transform: scale(1.05);
  }

  .saves > button {
    width: 100%;
  }

  .saves {
    display: flex;
    flex-grow: auto;
  }
</style>
