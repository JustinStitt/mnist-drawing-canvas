<script>
  import Canvas from "./lib/Canvas.svelte";
  let color = "#ffffff";
  let background = "#000";
</script>

<div class="container">
  <div class="title">MNIST Draw Pad</div>
  <Canvas {color} {background} />
  <div class="with-love">Made with 💖 by Justin</div>
</div>

<style>
  .title {
    width: 100%;
    position: absolute;
    top: 0;
    left: 0;
    background-color: #141313;
  }

  .with-love {
    position: absolute;
    left: 2%;
  }
</style>
