<script>
    import { onMount } from 'svelte';
  
    let fontLoaded = false;
  
    onMount(() => {
      const font = new FontFace('Zuume', 'url(/zuume.woff2)');
      font.load().then(() => {
        document.fonts.add(font);
        fontLoaded = true;
      });
    });
  </script>
  
  <div class="poster">
    <div class="banner">
      <h1 class:font-loaded={fontLoaded}>SMILE LIKE IT'S A WEDDING</h1>
    </div>
  </div>
  
  <style>
    @page {
      size: A4;
      margin: 0;
    }
  
    .poster {
      width: 210mm;
      height: 297mm;
      background-color: #FEFA99;
      position: relative;
      overflow: hidden;
      max-width: 100%;
      max-height: 100%;
      object-fit: contain;
    }
  
    .banner {
      width: 569px;
      height: 108px;
      background-color: #343233;
      display: flex;
      justify-content: center;
      align-items: center;
      position: absolute;
      top: 0;
      left: 50%;
      transform: translateX(-50%);
    }
  
    h1 {
      color: #FEFA99;
      font-size: 88px;
      letter-spacing: -4%;
      margin: 0;
      padding: 0;
      text-align: center;
      opacity: 0;
      transition: opacity 0.3s ease;
    }
  
    .font-loaded {
      font-family: 'Zuume', sans-serif;
      opacity: 1;
    }
  </style>