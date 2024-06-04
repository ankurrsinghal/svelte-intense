<script lang="ts">
  import "./styles.css";
  import SvelteIntense from "$lib/SvelteIntense.svelte";

  const IMAGE_ROOT = "/img/";
  const images = [
    {
      caption: "An annual month-long festival in Kyoto",
      className: "demo-image first",
      src: IMAGE_ROOT + "horse.jpg",
      thumbnailSrc: IMAGE_ROOT + "horse_thumb.jpg",
      title: "Gion Matsuri",
    },
    {
      caption: "Umbrellas are key!",
      className: "demo-image second",
      src: IMAGE_ROOT + "rain.jpg",
      thumbnailSrc: IMAGE_ROOT + "rain_thumb.jpg",
      title: "Rainy rain",
      vertical: true,
    },
    {
      caption: "Ancient Buddhist temple on a cliff",
      className: "demo-image third",
      src: IMAGE_ROOT + "temple.jpg",
      thumbnailSrc: IMAGE_ROOT + "temple_thumb.jpg",
      title: "Kiyomizu-dera",
    },
  ];
</script>

<section id="demos">
  <div class="gallery">
    {#each images as image}
      <SvelteIntense {...image} />
    {/each}
  </div>
</section>
