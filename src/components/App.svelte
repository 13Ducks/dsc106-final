<script>
  import Slideshow from "./Slideshow.svelte"; // Importing the Graph component
</script>

<main>
  <Slideshow />
</main>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap");

  :root {
    --color-bg: #ffffff;
    --color-outline: #c2c2c2;
    --color-shadow: hsl(0, 0%, 0%, 0.1);
    --color-text: #3f4252;
    --color-bg-1: hsla(0, 0%, 0%, 0.2);
    --color-shadow-1: hsl(0, 0%, 96%);
  }

  *,
  *::before,
  *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  main {
    text-align: center;
    font-family: "Nunito", sans-serif;
    font-weight: 300;
    font-size: 24px;
    color: var(--color-text);
  }
</style>
