<script lang="ts">
  import { onMount, onDestroy } from "svelte";

  interface ImageObject {
    x: number;
    y: number;
    width: number;
    height: number;
    scale: number;
    image: HTMLImageElement;
  }

  let imageUrl: string = "";
  let cropX: number = 0;
  let cropY: number = 0;
  let cropWidth: number = 200;
  let cropHeight: number = 200;
  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D | null;
  let imageObject: ImageObject | null = null;
  let animationFrameId: number;
  let windowRef: Window | null = null;

  function handleCrop(): void {
    if (ctx && imageObject) {
      const croppedCanvas: HTMLCanvasElement = document.createElement("canvas");
      croppedCanvas.width = cropWidth;
      croppedCanvas.height = cropHeight;
      const croppedCtx: CanvasRenderingContext2D | null =
        croppedCanvas.getContext("2d");
      if (croppedCtx) {
        const scaledCropX = (cropX - imageObject.x) / imageObject.scale;
        const scaledCropY = (cropY - imageObject.y) / imageObject.scale;
        const scaledCropWidth = cropWidth / imageObject.scale;
        const scaledCropHeight = cropHeight / imageObject.scale;

        croppedCtx.drawImage(
          imageObject.image,
          scaledCropX,
          scaledCropY,
          scaledCropWidth,
          scaledCropHeight,
          0,
          0,
          cropWidth,
          cropHeight,
        );
        const croppedImageUrl: string = croppedCanvas.toDataURL("image/jpeg");
        const link: HTMLAnchorElement = document.createElement("a");
        link.href = croppedImageUrl;
        link.download = "cropped_image.jpg";
        link.click();
      }
    }
  }

  function update(): void {
    // Update logic goes here (e.g., handling user input, animations)
  }

  function draw(): void {
    if (ctx && imageObject) {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(
        imageObject.image,
        imageObject.x,
        imageObject.y,
        imageObject.width,
        imageObject.height,
      );
      ctx.strokeStyle = "red";
      ctx.lineWidth = 2;
      ctx.strokeRect(cropX, cropY, cropWidth, cropHeight);
    }
  }

  function gameLoop(): void {
    update();
    draw();
    animationFrameId = requestAnimationFrame(gameLoop);
  }

  onMount(() => {
    windowRef = window;
    // drop event listener
    document.addEventListener("drop", (event) => {
      event.preventDefault();
      event.stopPropagation();
      const file: File | undefined = event.dataTransfer?.files?.[0];
      if (file) {
        imageUrl = URL.createObjectURL(file);
        const image: HTMLImageElement = new Image();
        image.src = imageUrl;
        image.onload = () => {
          const scale: number = Math.min(
            canvas.width / image.width,
            canvas.height / image.height,
          );
          imageObject = {
            x: (canvas.width - image.width * scale) / 2,
            y: (canvas.height - image.height * scale) / 2,
            width: image.width * scale,
            height: image.height * scale,
            scale,
            image,
          };
        };
      }
    });

    onDestroy(() => {
      if (animationFrameId) {
        cancelAnimationFrame(animationFrameId);
      }
      if (windowRef) {
        windowRef.removeEventListener("resize", handleResize);
      }
    });

    document.addEventListener("dragover", (event) => {
      event.preventDefault();
      event.stopPropagation();
    });

    canvas = document.getElementById("cropCanvas") as HTMLCanvasElement;
    ctx = canvas.getContext("2d");
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    window.addEventListener("resize", handleResize);
    gameLoop();
  });

  function handleResize(): void {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  }
</script>

<main class="bg-black overflow-hidden w-screen h-screen">
  <canvas id="cropCanvas" class="absolute top-0 left-0" />
  <div class="absolute bottom-0 left-0 bg-slate-900 text-white p-4">
    <label class="block mb-2">
      Crop X:
      <input
        type="number"
        bind:value={cropX}
        min="0"
        max={canvas?.width - cropWidth}
        class="w-20 text-black"
      />
    </label>
    <label class="block mb-2">
      Crop Y:
      <input
        type="number"
        bind:value={cropY}
        min="0"
        max={canvas?.height - cropHeight}
        class="w-20 text-black"
      />
    </label>
    <label class="block mb-2">
      Crop Width:
      <input
        type="number"
        bind:value={cropWidth}
        min="1"
        max={canvas?.width - cropX}
        class="w-20 text-black"
      />
    </label>
    <label class="block mb-2">
      Crop Height:
      <input
        type="number"
        bind:value={cropHeight}
        min="1"
        max={canvas?.height - cropY}
        class="w-20 text-black"
      />
    </label>
    <button
      on:click={handleCrop}
      class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
    >
      Crop Image
    </button>
  </div>
</main>
