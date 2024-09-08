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
      croppedCanvas.width = cropWidth / imageObject.scale;
      croppedCanvas.height = cropHeight / imageObject.scale;
      const croppedCtx: CanvasRenderingContext2D | null =
        croppedCanvas.getContext("2d");
      if (croppedCtx) {
        const scaledCropX = cropX / imageObject.scale;
        const scaledCropY = cropY / imageObject.scale;
        const scaledCropWidth = cropWidth / imageObject.scale;
        const scaledCropHeight = cropHeight / imageObject.scale;

        console.table(imageObject.image);
        console.table({
          x: imageObject.x,
          y: imageObject.y,
          width: imageObject.width,
          height: imageObject.height,
          scale: imageObject.scale,
        });

        console.table({
          cropX,
          cropY,
          cropWidth,
          cropHeight,
          scaledCropX,
          scaledCropY,
          scaledCropWidth,
          scaledCropHeight,
        });

        croppedCtx.drawImage(
          // original image
          imageObject.image,
          // crop area (x)
          scaledCropX,
          // crop area (y)
          scaledCropY,
          // crop area (width)
          scaledCropWidth,
          // crop area (height)
          scaledCropHeight,
          // destination (x)
          0,
          // destination (y)
          0,
          // destination (width)
          scaledCropWidth,
          // destination (height)
          scaledCropHeight,
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

  function reloadImage(image: HTMLImageElement): void {
    let scale: number = Math.min(
      canvas.width / image.width,
      canvas.height / image.height,
    );
    let scaledWidth: number = image.width * scale;
    let scaledHeight: number = image.height * scale;

    if (scaledHeight > image.height || scaledWidth > image.width) {
      scale = Math.min(
        canvas.width / image.width,
        canvas.height / image.height,
      );
    }

    imageObject = {
      x: (canvas.width - image.width * scale) / 2,
      y: (canvas.height - image.height * scale) / 2,
      width: image.width * scale,
      height: image.height * scale,
      scale,
      image,
    };

    cropWidth = image.width * scale;
    cropHeight = image.height * scale;
    recenterCrop();
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
      ctx.strokeStyle = "#fff";
      ctx.lineWidth = 1;
      ctx.strokeRect(cropX, cropY, cropWidth, cropHeight);
    }
  }

  function gameLoop(): void {
    update();
    draw();
    animationFrameId = requestAnimationFrame(gameLoop);
  }

  function recenterImage(): void {
    if (imageObject) {
      imageObject.x = (canvas.width - imageObject.width) / 2;
      imageObject.y = (canvas.height - imageObject.height) / 2;
    }
  }

  function recenterCrop(): void {
    cropX = (canvas.width - cropWidth) / 2;
    cropY = (canvas.height - cropHeight) / 2;
  }

  onMount(() => {
    windowRef = window;

    canvas = document.getElementById("cropCanvas") as HTMLCanvasElement;
    ctx = canvas.getContext("2d");

    window.addEventListener("resize", handleResize);
    handleResize();
    recenterCrop();
    gameLoop();

    // drag handles for crop area
    let isDragging: boolean = false;
    let dragHandle: string = "";
    let dragStartX: number = 0;
    let dragStartY: number = 0;
    const handlePadding: number = 16; // Padding around the drag handles

    canvas.addEventListener("mousedown", (event) => {
      const x: number = event.offsetX;
      const y: number = event.offsetY;
      if (
        x >= cropX - handlePadding &&
        x <= cropX + cropWidth + handlePadding &&
        y >= cropY - handlePadding &&
        y <= cropY + cropHeight + handlePadding
      ) {
        isDragging = true;
        dragStartX = x;
        dragStartY = y;
        if (x < cropX + handlePadding && y < cropY + handlePadding) {
          dragHandle = "top-left";
        } else if (
          x > cropX + cropWidth - handlePadding &&
          y < cropY + handlePadding
        ) {
          dragHandle = "top-right";
        } else if (
          x < cropX + handlePadding &&
          y > cropY + cropHeight - handlePadding
        ) {
          dragHandle = "bottom-left";
        } else if (
          x > cropX + cropWidth - handlePadding &&
          y > cropY + cropHeight - handlePadding
        ) {
          dragHandle = "bottom-right";
        } else if (x < cropX + handlePadding) {
          dragHandle = "left";
        } else if (x > cropX + cropWidth - handlePadding) {
          dragHandle = "right";
        } else if (y < cropY + handlePadding) {
          dragHandle = "top";
        } else if (y > cropY + cropHeight - handlePadding) {
          dragHandle = "bottom";
        } else {
          dragHandle = "move";
        }
      }
    });

    canvas.addEventListener("mousemove", (event) => {
      if (isDragging) {
        const x: number = event.offsetX;
        const y: number = event.offsetY;
        const dx: number = x - dragStartX;
        const dy: number = y - dragStartY;
        console.log(dragHandle);

        switch (dragHandle) {
          case "left":
            cropX += dx;
            cropWidth -= dx;
            break;
          case "right":
            cropWidth += dx;
            break;
          case "top":
            cropY += dy;
            cropHeight -= dy;
            break;
          case "bottom":
            cropHeight += dy;
            break;
          case "top-left":
            cropX += dx;
            cropY += dy;
            cropWidth -= dx;
            cropHeight -= dy;
            break;
          case "top-right":
            cropY += dy;
            cropWidth += dx;
            cropHeight -= dy;
            break;

          case "bottom-left":
            cropX += dx;
            cropWidth -= dx;
            cropHeight += dy;
            break;
          case "bottom-right":
            cropWidth += dx;
            cropHeight += dy;
            break;

          case "move":
            cropX += dx;
            cropY += dy;
            break;

          default:
            break;
        }
        dragStartX = x;
        dragStartY = y;
      }
    });

    canvas.addEventListener("mouseup", () => {
      isDragging = false;
    });

    // drop event listener
    document.addEventListener("drop", (event) => {
      event.preventDefault();
      const file: File | undefined = event.dataTransfer?.files?.[0];
      if (file) {
        imageUrl = URL.createObjectURL(file);
        const image: HTMLImageElement = new Image();
        image.src = imageUrl;
        image.onload = () => {
          reloadImage(image);
        };
      }
    });

    document.addEventListener("dragover", (event) => {
      event.preventDefault();
    });

    document.addEventListener("dragenter", (event) => {
      event.preventDefault();
    });

    document.addEventListener("dragleave", (event) => {
      event.preventDefault();
    });
  });

  onDestroy(() => {
    if (animationFrameId) {
      cancelAnimationFrame(animationFrameId);
    }
    if (windowRef) {
      windowRef.removeEventListener("resize", handleResize);
    }
  });

  function handleResize(): void {
    canvas.width = window.innerWidth - 200;
    canvas.height = window.innerHeight;
    // TODO: maybe delay & debounce this
    if (imageObject) {
      reloadImage(imageObject.image);
      recenterImage();
      // recenterCrop();
    }
  }

  function setCropAspectRatio(width: number, height: number): void {
    if (imageObject) {
      const isWidthLarger: boolean = width > height;
      const isHeightLarger: boolean = height > width;

      if (isWidthLarger) {
        cropHeight = (cropWidth / width) * height;
        cropWidth = cropWidth;
      } else if (isHeightLarger) {
        cropWidth = (cropHeight / height) * width;
        cropHeight = cropHeight;
      } else {
        cropWidth = cropWidth;
        cropHeight = cropHeight;
      }
      // recenterCrop();
    }
  }
</script>

<main class="bg-[#0f0f0f] overflow-hidden w-screen h-screen">
  <canvas id="cropCanvas" class="absolute top-0 left-0" />

  <section
    id="sidebar"
    class="w-[200px] max-w-[200px] absolute right-0 top-0 font-mono text-white bg-black bg-opacity-10 h-full p-2 flex justify-between items-center flex-col"
  >
    <div
      class="w-full transition-all hover:text-[#ff7700] font-bold text-right"
    >
      <p>
        <a href="https://x.com/alia0209">alịa</a>
      </p>
    </div>

    <div class="w-full grid gap-4">
      {#if imageObject}
        <div
          class="flex flex-col text-right text-white bg-gradient-to-bl from-[#0f0f0f] to-transparent p-4"
        >
          <p>{Math.round(cropWidth / (imageObject?.scale || 1))} W</p>
          <p>{Math.round(cropHeight / (imageObject?.scale || 1))} H</p>
          <p>{Math.round(cropX)} X</p>
          <p>{Math.round(cropY)} Y</p>

          <!-- aspect ratio as x:y simplified (i.e 300:200 -> 3:2)-->
          <p>
            {Math.round(cropWidth / (imageObject?.scale || 1)) /
              Math.round(cropHeight / (imageObject?.scale || 1))}
          </p>
        </div>
        <div class="bg-opacity-40 bg-black p-2 w-full">
          <p class="pb-2">Aspect Ratio</p>

          <div class="grid grid-cols-2 gap-2">
            {#each [[16, 9], [4, 3], [1, 1], [3, 4], [9, 16]] as [w, h]}
              <button
                on:click={() => setCropAspectRatio(w, h)}
                class="hover:bg-[#ff7700] p-2 transition-all rounded overflow-hidden flex flex-col justify-center items-center bg-[#0f0f0f] z-10"
              >
                {w}:{h}
              </button>
            {/each}
          </div>
        </div>
        <button
          on:click={handleCrop}
          class="group p-2 w-full transition-all rounded overflow-hidden hover:invert invert-0 flex justify-between items-center bg-[#0f0f0f] z-10"
        >
          <p>download</p>
          <img src="/floppy-disk.svg" alt="save" class="w-8 h-8" />
        </button>
      {/if}
    </div>
  </section>
</main>
