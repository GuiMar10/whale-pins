<script lang="ts">
  var pins: string[] = [];
  var link = "";
  var color = "#ffffff";
  import Masonry from "../lib/Masonry.svelte";
  function getBg(H: string) {
    // Convert hex to RGB first
    let r: any = 0,
      g: any = 0,
      b: any = 0;
    if (H.length == 4) {
      r = "0x" + H[1] + H[1];
      g = "0x" + H[2] + H[2];
      b = "0x" + H[3] + H[3];
    } else if (H.length == 7) {
      r = "0x" + H[1] + H[2];
      g = "0x" + H[3] + H[4];
      b = "0x" + H[5] + H[6];
    }
    // Then to HSL
    r /= 255;
    g /= 255;
    b /= 255;
    let cmin = Math.min(r, g, b),
      cmax = Math.max(r, g, b),
      delta = cmax - cmin,
      h = 0,
      s = 0,
      l = 0;

    if (delta == 0) h = 0;
    else if (cmax == r) h = ((g - b) / delta) % 6;
    else if (cmax == g) h = (b - r) / delta + 2;
    else h = (r - g) / delta + 4;

    h = Math.round(h * 60);

    if (h < 0) h += 360;

    l = (cmax + cmin) / 2;
    s = delta == 0 ? 0 : delta / (1 - Math.abs(2 * l - 1));
    s = +(s * 100).toFixed(1);
    l = +(l * 100).toFixed(1);

    return "hsl(" + h + "," + (s - 20) + "%," + 8 + "%)";
  }
  function transformYouTubeLink(link: string) {
    const url = new URL(link);
    const videoId = url.searchParams.get("v");
    return `https://www.youtube.com/embed/${videoId}`;
  }
  function getType(link: string) {
    var type = "";
    if (link.includes("https://")) {
      type = "link";
    }
    if (/\p{Emoji}/u.test(link) && link.length == 2) {
      type = "emoji";
    }
    if (link.includes("youtube.com/watch?v=")) {
      type = "yt";
    }
    if (isImage(link)) {
      type = "image";
    }
    if (type == "") {
      type = "note";
    }
    return type;
  }
  function setPin(url = link) {
    if (!url) return;
    if (url.includes(".") && !url.startsWith("http")) {
      url = "https://" + url;
    }
    pins = [...pins, url];
    link = "";
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
      setPin(data);
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
        url.includes("auto=format") ||
        url.includes("picsum")
    );
  }
  function populate() {
    pins = Array(10)
      .fill(0)
      .map(
        (_, i) =>
          `https://picsum.photos/${500 - Math.floor(Math.random() * 20 + 1)}/700`
      );
  }
  async function emojiMetadata(emoji: string) {
    const em = await fetch(
      `https://emoji-api.com/emojis?search=${emoji}&access_key=20fa1d1b04d5a670f8f7a0d1ace322c0d49f9012`
    )
      .then((e) => e.json())
      .then((e) => {
        return e;
      });
    const emName = em[0].unicodeName.split(" ").slice(1).join(" ");

    return capitalizeFirstLetter(emName);
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
  href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200&icon_names=ios_share,search"
/>
<link
  href="https://fonts.googleapis.com/css2?family=Inria+Serif:ital,wght@0,300;0,400;0,700;1,300;1,400;1,700&family=Nunito:ital,wght@0,200..1000;1,200..1000&family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap"
  rel="stylesheet"
/>
<input
  on:keydown={(e) => {
    if (e.key === "Enter") {
      setPin();
    }
  }}
  placeholder="Add pin..."
  bind:value={link}
  id="link"
/>
<button id="pin" on:click={() => setPin()}>+</button>
<!--<div id="top" role="region" on:drop={handleDrop}>to top</div>-->
<div role="region" on:drop={handleDrop} on:dragover={handleDragOver}>
  <Masonry reset={false} colWidth="minmax(Min(12em, 100%), 1fr);">
    {#each pins as pin, index}
      {#if getType(pin) == "yt"}
        <div class="embed">
          <iframe
            src={transformYouTubeLink(pin)}
            frameborder="0"
            title="YouTube video"
          >
          </iframe>
        </div>
      {:else if getType(pin) == "image"}
        <a href={pin} id={"i" + index}>
          <img id="pinned" src={pin} alt="A" />
        </a>
      {:else if getType(pin) == "link"}
        <a style="text-decoration: none !important;" href={pin}>
          <div class="pin">
            <img
              src={getFavicon(pin)}
              alt="Favicon"
              style="width: 15px; height: 15px; border-radius: 50%;"
            />
            <span>{siteName(pin)}</span>
            <br />
            <span style="opacity: 0.5"> {pin}</span>
          </div>
        </a>
      {:else if getType(pin) == "note"}
        <div class="pin" id="note" contenteditable="true">
          {pin}
        </div>
      {:else if getType(pin) == "emoji"}
        <div class="pin" id="emoji">
          {pin}
          <br />
          {#await emojiMetadata(pin) then name}
            <span style="font-family: Roboto; font-size: 14px; opacity: 0.4;"
              >{name}</span
            >
          {/await}
        </div>
      {/if}
    {/each}
  </Masonry>
  {#if pins.length == 0}
    <div
      style="color: white; text-align: center; margin-top: 20px; opacity: 0.3;"
    >
      <p>Nothing here yet!</p>
      <p>Click the + button to add a link</p>
    </div>
  {/if}
  <div id="tabs">
    <a href="/" on:click={populate} id="selected">inspo</a>
    <a href="/" on:click={() => (pins = [])}>new</a>
  </div>
  {#if pins.length > 0}
    <div style="position: fixed; bottom: 20px; right: 20px;">
      <input
        type="color"
        bind:value={color}
        on:input={() => {
          document.documentElement.style.setProperty("--primaryColor", color);
          document.documentElement.style.setProperty(
            "--background",
            getBg(color)
          );
        }}
      />
      <button id="share">
        <span class="material-icons-outlined">ios_share</span>
      </button>
    </div>
  {/if}
</div>

<style lang="scss">
  @font-face {
    font-family: "Emoji";
    src: url("/AppleColorEmoji.ttf") format("truetype");
  }
  :root {
    color-scheme: dark;
    --background: #171717;
    background-color: var(--background);
    font-family: Roboto;
    --primaryColor: rgb(255, 255, 255);
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
    grid-template-columns: repeat(auto-fill, minmax(140px, 1fr));
    gap: 8px;
    margin-top: 10px;
    grid-auto-rows: minmax(40px, 100%);
  }
  input[type="color"] {
    transform: translateY(4px);
    margin-right: 2px;
  }
  input[type="color"]::-moz-color-swatch,
  input[type="color"]::-webkit-color-swatch-wrapper,
  input[type="color"]::-webkit-color-swatch {
    border-radius: 100px;
    border-style: none;
  }

  button#share,
  input[type="color"] {
    border: 0;
    border-radius: 100px;
    padding: 4px;
    width: 35px;
    cursor: pointer;
    height: 35px;
    transition: 0.1s;
    animation: pincreate 0.3s;
    backdrop-filter: blur(10px);
    box-shadow: 0px 0px 5px 1px rgba(0, 0, 0, 0.5);
    background-color: rgba(59, 59, 59, 0.7);
    &:hover {
      scale: 1.03;
      background-color: rgba(58, 58, 58, 0.8);
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
  input#link {
    width: 100%;
    background-color: transparent;
    color: white;
    height: 45px;
    border: none;
    font-size: 1.2rem;
    outline: none;
    font-family: Inria Serif;
    font-style: italic;
    padding: 4px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
    margin-top: 6px;
    transition: 0.2s;
    &:focus {
      border-color: rgba(255, 255, 255, 0.1);
      &::placeholder {
        opacity: 0.5;
      }
    }
    &::placeholder {
      opacity: 0.3;
    }
  }
  div#tabs {
    position: fixed;
    bottom: 20px;
    left: 50%;
    box-shadow: 0px 0px 5px 1px rgba(0, 0, 0, 0.3);
    transform: translateX(-50%);
    background-color: rgba(60, 60, 60, 0.8);
    border: 2px solid rgba(255, 255, 255, 0.05);
    backdrop-filter: blur(12px);
    border-radius: 16px;
    user-select: none;
    padding: 6px;
    /*& input {
      border-radius: 12px;
      height: 20px;
      border: 1px solid rgba(255, 255, 255, 0.1);
      outline: 0;
      padding: 5px 8px;
      background: transparent;
      transition: 0.1s;
      width: 70px;
      &:focus {
        width: 100px;
      }
    }*/
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
  div#emoji {
    cursor: pointer;
    font-size: 24px;
    padding: 8px;
    font-family: "Emoji";
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
    max-width: 100%;
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
</style>
