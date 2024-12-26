<script lang="ts">
  var pins: { link: string; title?: string }[] = [];
  var link = "";
  var color = "";
  function transformYouTubeLink(link: string) {
    const url = new URL(link);
    const videoId = url.searchParams.get("v");
    return `https://www.youtube.com/embed/${videoId}`;
  }
  function setPin() {
    if (link === "") return;
    if (link.includes(".") && !link.startsWith("http")) {
      link = "https://" + link;
    }
    pins = [...pins, { link: link }];
    link = "";
    console.log(pins);
  }
  function getFavicon(url: string) {
    return `https://www.google.com/s2/favicons?domain=${url}`;
  }
  function capitalizeFirstLetter(str: string) {
    return str[0].toUpperCase() + str.slice(1);
  }
  function siteName(url: string) {
    return capitalizeFirstLetter(
      url.replace("https://", "").split("/")[0].split(".")[0]
    );
  }
  function handleDrop(event: DragEvent) {
    event.preventDefault();
    const data = event.dataTransfer?.getData("text/plain");
    if (data) {
      pins = [...pins, { link: data }];
    }
  }
  function handleDragOver(event: DragEvent) {
    event.preventDefault();
  }
  function isImage(url: string) {
    const imageExtensions = ["png", "jpg", "jpeg", "webp", "gif"];
    return imageExtensions.some(
      (ext) =>
        url.includes("." + ext) ||
        url.includes("?format=" + ext) ||
        url.includes("blobcdn.") ||
        url.includes("auto=format")
    );
  }
</script>

<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="" />
<link
  href="https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,200..1000;1,200..1000&family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap"
  rel="stylesheet"
/>
<link
  rel="stylesheet"
  href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200&icon_names=ios_share"
/>
<input
  on:keydown={(e) => {
    if (e.key === "Enter") {
      setPin();
    }
  }}
  placeholder="Link..."
  bind:value={link}
  id="link"
/>
<button id="pin" on:click={() => setPin()}>+</button>
<!--<div id="top" role="region" on:drop={handleDrop}>to top</div>-->
<div
  id="container"
  role="region"
  on:drop={handleDrop}
  on:dragover={handleDragOver}
>
  {#each pins as pin, index}
    {#if pin.link.includes("youtube.com") && pin.link.includes("watch")}
      <div class="embed">
        <iframe
          src={transformYouTubeLink(pin.link)}
          frameborder="0"
          title="YouTube video"
        >
        </iframe>
      </div>
    {:else if isImage(pin.link)}
      <a href={pin.link}>
        <img id="pinned" src={pin.link} alt="A" />
      </a>
    {:else if pin.link.includes("https://")}
      <a style="text-decoration: none !important;" href={pin.link}>
        <div class="pin">
          <img
            src={getFavicon(pin.link)}
            alt="Favicon"
            style="width: 15px; height: 15px; border-radius: 50%;"
          />
          <span>{siteName(pin.link)}</span>
          <br />
          <span style="opacity: 0.5"> {pin.link}</span>
        </div>
      </a>
    {:else}
      <div class="pin" id="note" contenteditable="true">
        {pin.link}
      </div>
    {/if}
  {/each}
</div>
{#if pins.length == 0}
  <div
    role="region"
    on:drop={handleDrop}
    on:dragover={handleDragOver}
    style="color: white; text-align: center; margin-top: 20px; opacity: 0.3;,"
  >
    <p>Nothing here yet!</p>
    <p>Click the + button to add a link</p>
  </div>
{/if}
<div id="tabs">
  <a href="/" id="selected">head</a>
  <a href="/">links</a>
</div>
{#if pins.length > 0}
  <div style="position: fixed; bottom: 20px; right: 20px;">
    <input
      type="color"
      bind:value={color}
      on:input={() =>
        document.documentElement.style.setProperty("--primaryColor", color)}
    />
    <button id="share">
      <span class="material-icons-outlined">ios_share</span>
    </button>
  </div>
{/if}

<style lang="scss">
  :root {
    color-scheme: dark;
    background-color: #1c1b22;
    font-family: Roboto;
    --primaryColor: orange;
    --darker: darkorange;
  }
  * {
    font-family: Roboto;
  }
  .material-icons-outlined {
    font-family: "Material Symbols Outlined" !important;
    font-size: 18px;
  }
  #container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 8px;
    margin-top: 10px;
    grid-auto-rows: minmax(40px, 100%);
  }
  input[type="color"] {
    border: 0;
    border-radius: 100px;
    padding: 4px;
    width: 35px;
    height: 35px;
    cursor: pointer;
    transition: 0.1s;
    transform: translateY(4px);
    margin-right: 2px;
    animation: pincreate 0.3s;
  }
  input[type="color"]::-moz-color-swatch,
  input[type="color"]::-webkit-color-swatch-wrapper,
  input[type="color"]::-webkit-color-swatch {
    border-radius: 100px;
    border-style: none;
  }

  button#share {
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 100px;
    padding: 4px;
    width: 35px;
    cursor: pointer;
    height: 35px;
    transition: 0.1s;
    animation: pincreate 0.3s;
    &:hover {
      scale: 1.03;
      background-color: rgba(255, 255, 255, 0.1);
    }
    &:active {
      scale: 0.97;
    }
  }
  button#pin {
    background-color: var(--primaryColor);
    border: 0;
    border-radius: 100px;
    padding: 2px;
    color: black;
    cursor: pointer;
    font-size: 25px;
    width: 35px;
    position: absolute;
    top: 15px;
    right: 10px;
    height: 35px;
    transition: 0.2s;
    &:hover {
      background-color: var(--darker);
      scale: 1.05;
      box-shadow: 0px 0px 5px 1px rgba(255, 187, 0, 0.5);
    }
    &:active {
      scale: 0.95;
    }
  }
  #top {
    font-size: 0.8rem;
    padding: 10px;
    color: rgba(255, 255, 255, 0.3);
    border: 1px solid rgba(0, 255, 255, 0.5);
    background: rgba(0, 255, 255, 0.1);
    border-radius: 16px;
    text-align: center;
    margin-top: 20px;
    animation: topcolor 2s infinite;
    height: 5px;
    transition: 0.2s;
    opacity: 0.5;
    &:-moz-drag-over {
      height: 100px;
      opacity: 1;
    }
  }
  input#link {
    width: 100%;
    background-color: transparent;
    color: white;
    height: 45px;
    border: none;
    font-size: 1.2rem;
    outline: none;
    padding: 4px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
    margin-top: 6px;
    transition: 0.2s;
    &:focus {
      border-color: rgba(255, 255, 255, 0.1);
    }
  }
  div#tabs {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    background-color: rgba(60, 60, 60, 0.6);
    backdrop-filter: blur(10px);
    border-radius: 16px;
    user-select: none;
    padding: 6px;
    & a {
      &#selected {
        background-color: var(--primaryColor);
        color: black;
      }
      display: inline-block;
      background-color: rgba(255, 255, 255, 0.1);
      text-decoration: none;
      padding: 5px 8px;
      border-radius: 12px;
      color: white;
      transition: 0.1s;
      height: 20px;
      &:hover {
        scale: 0.97;
        background-color: rgba(255, 255, 255, 0.2);
      }
      &:active {
        scale: 0.9;
        opacity: 0.3;
      }
    }
  }
  div.pin {
    background-color: rgba(24, 24, 24, 0.2);
    border: 2px solid rgba(128, 128, 128, 0.2);
    border-radius: 10px;
    color: white;
    padding: 6px;
    backdrop-filter: blur(10px);
    transition: 0.1s;
    height: fit-content;
    animation: pincreate 0.4s;
    &:hover {
      scale: 1.02;
      rotate: 1deg;
    }
    &:active {
      scale: 0.98;
    }
  }
  img#pinned {
    border-radius: 8px;
    display: block;
    width: 100%;
    overflow: hidden; /* Hides overflowing content */
    text-overflow: ellipsis; /* Adds ellipsis for text */
    white-space: nowrap; /* Prevents text wrapping */
    position: relative;
    cursor: pointer;
    transition: 0.1s;
    animation: pincreate 0.4s;
    &:hover {
      scale: 0.99;
      opacity: 0.8;
    }
    &:active {
      scale: 0.98;
      opacity: 0.6;
    }
  }
  .embed {
    width: 100%;
    iframe {
      width: 100%;
      border-radius: 8px;
    }
  }
  @keyframes pincreate {
    from {
      scale: 0.95;
      opacity: 0.2;
    }
  }
  @keyframes topcolor {
    30% {
      background-color: rgba(255, 251, 0, 0.1);
      border-color: rgba(255, 251, 0, 0.6);
    }
    50% {
      background-color: rgba(255, 0, 242, 0.1);
      border-color: rgba(255, 0, 234, 0.6);
    }
  }
</style>
