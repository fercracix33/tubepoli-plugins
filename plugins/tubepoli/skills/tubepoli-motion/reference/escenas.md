# Contrato técnico de una escena

Una escena es el **cuerpo** de una subcomposición de HyperFrames: marcado + `<style>` + `<script>` en línea. tubepoli pone el `<template>`, la raíz con su tamaño (1920×1080 por defecto, `position: absolute; inset: 0; overflow: hidden`), el reloj y el ayudante `tp`. No escribas `<html>`, `<head>`, `<body>`, `<template>` ni `data-composition-id`.

## Tiempo

- **CSS `@keyframes` y Web Animations** se sincronizan solos con el tiempo **local** de la escena (0 = cuando empieza). Usa `animation-delay` para escalonar y `animation-fill-mode: both`.
- **Canvas o SVG por código**: registra una función de dibujo con `tp.scene((t, root) => { … })`. `t` son los segundos locales; `root` es la raíz de la escena: busca tus elementos con `root.querySelector` dentro de la función (no uses `document.currentScript`). Dibuja el estado completo de ese instante: el render salta de fotograma en fotograma y la función puede llamarse en cualquier orden.
- **Prohibido** (el lint lo rechaza): `Date`, `performance.now`, `requestAnimationFrame`, `setTimeout`/`setInterval`, `Math.random`/`crypto`, `.play()`, red (`fetch`, `XMLHttpRequest`, `WebSocket`, `import()`), `<script src>`, `<iframe>`, almacenamiento y GSAP. Las escenas no llevan `<audio>`.
- Azar con semilla: `const rnd = tp.random(42); rnd()` da siempre la misma serie.
- Curvas: `tp.ease.out(x)`, `tp.ease.inOut(x)`, `tp.ease.clamp(x, a, b)`.

## Marca del canal

Variables CSS: `var(--tp-primary)`, `var(--tp-secondary)`, `var(--tp-accent)`, `var(--tp-text)`, `var(--tp-bg)`, `var(--tp-font)`. En JS: `tp.brand(root)` devuelve `{primary, secondary, accent, text, background, font}`. Pon siempre un valor de reserva: `var(--tp-primary, #ff3b30)`.

## Ejemplo 1 — titular cinético (CSS)

```html
<div class="hl">
  <span style="--i:0">Tres errores</span>
  <span style="--i:1">que matan</span>
  <span style="--i:2">tu intro</span>
</div>
<style>
  .hl { position:absolute; inset:0; display:flex; flex-direction:column; justify-content:center; padding:0 160px;
        background:var(--tp-bg, #0b0b0f); font-family:var(--tp-font, Inter), sans-serif; }
  .hl span { display:block; font-size:120px; font-weight:800; line-height:1.05; color:var(--tp-text, #fff);
             opacity:0; transform:translateY(40px);
             animation:up .6s cubic-bezier(.2,.7,.2,1) calc(.25s + var(--i) * .35s) both; }
  .hl span:last-child { color:var(--tp-primary, #ff3b30); }
  @keyframes up { to { opacity:1; transform:none; } }
</style>
```

## Ejemplo 2 — dato con barra (SVG + CSS)

```html
<svg viewBox="0 0 1920 1080" class="dato">
  <text x="160" y="520" class="num">73 %</text>
  <text x="160" y="610" class="lbl">de los canales no cuida su intro</text>
  <rect x="160" y="680" width="1600" height="28" rx="14" class="bg"/>
  <rect x="160" y="680" width="1168" height="28" rx="14" class="fill"/>
</svg>
<style>
  .dato { position:absolute; inset:0; width:100%; height:100%; background:var(--tp-bg, #0b0b0f); }
  .num { font:800 220px var(--tp-font, Inter), sans-serif; fill:var(--tp-primary, #ff3b30); }
  .lbl { font:500 56px var(--tp-font, Inter), sans-serif; fill:var(--tp-text, #fff); }
  .bg { fill:rgba(255,255,255,.12); }
  .fill { fill:var(--tp-primary, #ff3b30); transform-box:fill-box; transform-origin:left; transform:scaleX(0);
          animation:grow 1.2s cubic-bezier(.2,.7,.2,1) .4s both; }
  @keyframes grow { to { transform:scaleX(1); } }
</style>
```

## Ejemplo 3 — partículas que forman una línea (canvas)

```html
<canvas width="1920" height="1080" class="cv"></canvas>
<style>.cv { position:absolute; inset:0; width:100%; height:100%; }</style>
<script>
  const rnd = tp.random(7);
  const pts = Array.from({ length: 160 }, (_, i) => ({ x: rnd() * 1920, y: rnd() * 1080, tx: 160 + i * 10, ty: 540 }));
  tp.scene((t, root) => {
    const ctx = root.querySelector(".cv").getContext("2d");
    const b = tp.brand(root);
    const k = tp.ease.inOut(tp.ease.clamp(t / 2));
    ctx.fillStyle = b.background || "#0b0b0f";
    ctx.fillRect(0, 0, 1920, 1080);
    ctx.fillStyle = b.primary || "#ff3b30";
    for (const p of pts) {
      ctx.beginPath();
      ctx.arc(p.x + (p.tx - p.x) * k, p.y + (p.ty - p.y) * k, 5, 0, Math.PI * 2);
      ctx.fill();
    }
  });
</script>
```

## Comprobación antes de dar una escena por buena

- `lintScene` sin errores (los avisos, léelos: animación infinita, easing con rebote).
- Tira de fotogramas: el primer y el último fotograma tienen sentido (sin pantallas vacías salvo que se busque), ningún texto cortado, solapado o pegado al borde.
- Hoja 4×3 si la escena dura más de 5 s.
- Duración de la escena = la del beat del storyboard (o actualiza el storyboard).
