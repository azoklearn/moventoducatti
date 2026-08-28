# PROMPT — Site cinématique Ducati Superleggera V4 (scroll-scrubbing vidéo)

Crée un site one-page cinématique présentant la Ducati Superleggera V4, construit autour d'une vidéo de fond dont la lecture est pilotée par le scroll. Livre un SEUL fichier `index.html` complet, prêt à lancer. Stack imposée : HTML + CSS + JavaScript vanilla, GSAP 3.12.5 + ScrollTrigger via CDN jsdelivr, polices Google Fonts **Anton** (display) + **Inter** (corps). AUCUN framework, AUCUNE lib de smooth-scroll (pas de Lenis), aucun fichier séparé.

## INTERDITS ABSOLUS
- Pas d'`autoplay` ni de `controls` sur la vidéo (sauf fallback iOS décrit plus bas).
- Ne JAMAIS assigner `video.currentTime` directement dans un handler de scroll.
- Aucun coin arrondi, aucune ombre portée, aucun dégradé décoratif, aucun emoji.
- Texte réel dans le DOM (jamais généré en JS), sauf le chiffre du compteur.

## VIDÉO
- Source UNIQUE de la balise `<video>`, à utiliser telle quelle (fichier déjà hébergé, H.264 1080p, images-clés denses, Range et CORS activés — ne pas la remplacer, ne pas la ré-encoder) :
  `https://res.cloudinary.com/dk2kai0as/video/upload/v1787946182/videoducatti_ntx5mb.mp4` (type video/mp4)
- Pour information si la vidéo devait un jour être remplacée : le mp4 doit être en H.264 avec une image-clé toutes les 6 frames, sinon le scrubbing se fige. Conversion :
  `ffmpeg -i source.webm -c:v libx264 -pix_fmt yuv420p -crf 21 -g 6 -keyint_min 6 -sc_threshold 0 -an -movflags +faststart videoducatti.mp4`
- Attributs : `muted playsinline webkit-playsinline preload="auto" disablepictureinpicture`.
- CSS : `position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; z-index: 0`, dans une scène `position: sticky; top: 0; height: 100vh; height: 100svh; overflow: hidden`, elle-même dans une piste `#track` de **600vh** (400vh sous 768px).

## MÉCANIQUE DE SCRUBBING (au pixel près)
- ScrollTrigger : `trigger: #track, start: "top top", end: "bottom bottom", scrub: true`.
- Dans `onUpdate` : `targetTime = Math.min(progress * video.duration, video.duration - 0.05)` (borne sous la durée pour éviter l'état "ended"). Ne rien assigner d'autre ici.
- Boucle `requestAnimationFrame` séparée avec lerp : `currentTime += (targetTime - currentTime) * 0.1`.
  - La boucle s'arrête quand `|diff| < 0.001` ET `!video.seeking` ; elle est relancée par `onUpdate` si éteinte (garder l'id rAF à null quand stoppée).
  - N'assigner `video.currentTime` QUE si `video.readyState >= 2` ET `!video.seeking` (ne jamais empiler un seek sur un seek en cours — c'est ce qui fige l'image).
- Toute l'initialisation attend l'événement `loadedmetadata` (ou `readyState >= 1` si déjà passé), puis `ScrollTrigger.refresh()`.

## PRÉCHARGEMENT
Overlay fixe noir plein écran z-index 100 : « DUCATI » en Anton (clamp(2.2rem, 6vw, 3.6rem)), barre de 180×2px (fond blanc à 14 %) avec remplissage rouge #CC0000 alimenté par l'événement `progress` (`buffered.end(dernier)/duration`), sous-titre « SUPERLEGGERA V4 » en 11px espacé 0.38em gris. Sortie : fondu CSS 600ms déclenché par `canplaythrough`, élément retiré du DOM 700ms après. Garde-fou : `setTimeout` de 8s qui force la fermeture.

## LES 7 BLOCS DE TEXTE (timeline maîtresse)
Blocs en `position: absolute; inset: 0; z-index: 2; display: flex; align-items: center; pointer-events: none`, contenu dans `.inner` (max-width 560px). Une timeline GSAP unique de **100 unités** (`defaults: {ease:"none"}`, même ScrollTrigger scrubbed que ci-dessus), terminée par `tl.set({}, {}, 100)` pour caler la durée à exactement 100.
- Entrée de chaque bloc (sauf hero, visible d'emblée via `gsap.set` autoAlpha 1) : `fromTo {autoAlpha: 0, y: 30} → {autoAlpha: 1, y: 0, duration: 3}` positionnée à `start + 0.5`.
- Sortie (sauf outro, qui reste) : `to {autoAlpha: 0, filter: "blur(8px)", duration: 3}` positionnée à `end - 1.5`.
- Plages : hero **0–10**, moteur **12–25**, puissance **28–42**, poids **45–58**, aéro **60–72**, carbone **75–88**, outro **90–100**. Jamais deux blocs visibles simultanément.
- L'indicateur de scroll du hero disparaît via `tl.to(".scroll-hint", {autoAlpha: 0, duration: 2}, 2)`.

### Contenus EXACTS (verbatim, en français)
1. **HERO** (centré) — h1 « Superleggera V4 » en clamp(3rem, 12vw, 11rem), SANS white-space:nowrap (il doit pouvoir se couper en deux lignes) ; sous-titre « Ducati. 100 anni. » 13px espacé 0.44em ; en bas, hint « SCROLL » 10px + trait vertical 1×52px avec balayage blanc animé (keyframes 1.8s cubic-bezier(0.65,0,0.35,1) infini).
2. **LE MOTEUR** (aligné à gauche, .inner max-width 40%) — kicker « Le moteur » ; h2 « Desmosedici Stradale R » ; lead « Dérivé directement de la MotoGP, le V4 le plus abouti jamais monté sur une Ducati de série. Distribution desmodromique, régime maximal de 15 500 tr/min. » ; ligne spec « 998 cm³ · V4 90° · vilebrequin contre-rotatif ».
3. **LA PUISSANCE** (centré) — kicker « La puissance » ; compteur animé de 0 à 224 en Anton clamp(5rem, 18vw, 15rem), `font-variant-numeric: tabular-nums`, unité « CV » à 0.22em en rouge #CC0000 ; ligne spec « 234 cv avec le kit racing Akrapovič ». Le compteur est un objet `{v: 0}` tweené vers 224, position 31, duration 9, `onUpdate` qui écrit `Math.round(v)` dans le span.
4. **LE POIDS** (centré) — kicker « Le poids » ; h2 « 152,2 kg à sec » ; lead « La moto de série la plus légère jamais produite par Ducati. » ; ligne spec « Ratio poids / puissance : 0,68 kg par cv ».
5. **L'AÉRODYNAMIQUE** (aligné à droite, texte à droite) — kicker « L'aérodynamique » ; h2 « Ailerons biplan » ; lead « Directement dérivés de la Desmosedici GP16, les ailerons en carbone génèrent l'appui le plus élevé jamais atteint sur une moto homologuée. » ; ligne spec « 50 kg d'appui à 270 km/h ».
6. **LE CARBONE** (centré) — kicker « Le carbone » ; h2 « Fibre intégrale » ; liste (min-width min(460px, 100%)) : « Cadre monocoque −1,7 kg / Sous-cadre −1,2 kg / Bras oscillant −0,9 kg / Jantes −3,4 kg / Carénages −1,8 kg ». Chaque li : flex space-between, `border-top: 1px solid rgba(255,255,255,0.22)` (+ border-bottom sur le dernier), 12px, graisse 500, espacement 0.24em, uppercase, valeur à droite en gris.
7. **OUTRO** (centré) — h2 « 500 exemplaires<br>dans le monde » ; lead « Chaque machine est numérotée, livrée avec son certificat d'authenticité et l'accès au programme SBK Experience. » ; bouton « Découvrir ».

## DIRECTION ARTISTIQUE (valeurs exactes)
- Palette : noir #000000, blanc #FFFFFF, rouge #CC0000 (accent unique et rare — jamais en texte de petite taille, contraste oblige). Gris secondaire rgba(255,255,255,0.62). Hairline rgba(255,255,255,0.22). Lead rgba(255,255,255,0.78).
- Titres (h1, h2) : Anton, uppercase, `letter-spacing: -0.03em; line-height: 0.9; font-weight: 400`. h2 : clamp(2rem, 5.4vw, 4.6rem).
- Kickers : 11px, graisse 600, espacement 0.38em, uppercase, gris, margin-bottom 18px, précédés d'un tiret rouge 28×2px inline (inversé côté droit pour le bloc aéro, masqué sur les blocs centrés et en mobile).
- Padding horizontal global : `clamp(24px, 6vw, 96px)`.
- Voile de lisibilité sur la vidéo (z-index 1) : `radial-gradient(ellipse 120% 90% at 50% 48%, rgba(0,0,0,0.55) 0%, rgba(0,0,0,0.18) 62%, rgba(0,0,0,0.38) 100%)` + `linear-gradient(to top, rgba(0,0,0,0.62) 0%, transparent 28%)`.
- Grain de film : `body::after` fixe plein écran, z-index 8, `opacity: 0.04; mix-blend-mode: overlay`, SVG inline en data-URI avec `feTurbulence type="fractalNoise" baseFrequency="0.9" numOctaves="2" stitchTiles="stitch"` sur un rect 300×300.
- Barre de progression : div fixe 2px en haut, rouge, `transform: scaleX(0); transform-origin: left`, z-index 10, pilotée par un `gsap.quickSetter(..., "scaleX")` dans le onUpdate d'un ScrollTrigger sur la piste.
- Header fixe (z-index 9, pointer-events none) : « DUCATI » en Anton 15px à gauche, « SUPERLEGGERA V4 · 001 / 500 » 11px espacé 0.32em gris à droite, padding 22px.
- CTA : `border: 1px solid rgba(255,255,255,0.45); padding: 17px 54px; font: 600 12px Inter; letter-spacing: 0.38em; uppercase; background: transparent; pointer-events: auto; overflow: hidden`. Remplissage au survol via `::before` rouge en `transform: scaleX(0→1); transform-origin: left; transition: 420ms cubic-bezier(0.65,0,0.35,1)`, texte dans un span z-index 1, bordure qui passe au rouge.

## FALLBACK iOS (obligatoire)
Détection : `/iPad|iPhone|iPod/.test(navigator.userAgent) || (navigator.platform === "MacIntel" && navigator.maxTouchPoints > 1)`. Dans ce cas : classe `.ios` sur `<html>` ; la vidéo passe en `loop` + `play()` muted (rattrapage sur premier `touchstart` si l'autoplay est refusé) ; AUCUN scrubbing ; compteur figé à « 224 » ; un ScrollTrigger sur la piste toggle une classe `.on` sur chaque bloc selon les mêmes plages de % ; le CSS sous `.ios` fait le fondu classique : état par défaut `opacity: 0; translateY(30px); blur(8px)`, état `.on` tout à zéro, `transition: 650ms ease` sur les trois propriétés.

## ACCESSIBILITÉ / PERF / RESPONSIVE
- `prefers-reduced-motion: reduce` : ne rien initialiser de GSAP ; classe `.static` sur `<html>` : piste en hauteur auto, vidéo statique de 62vh (première image = poster), tous les blocs en flux normal (padding 64px, border-top hairline, opacité 1 forcée), loader/hint/barre masqués. Dupliquer ces règles dans un `<noscript><style>` pour le cas sans JS.
- `will-change: transform, opacity, filter` appliqué UNIQUEMENT via une classe togglée par des ScrollTriggers numériques par bloc (plage étendue de ±4 %), retirée hors plage.
- Sous 768px : piste 400vh, tout centré pleine largeur, padding 24px, h1 clamp(2.6rem, 15vw, 5rem), h2 clamp(1.8rem, 9vw, 3rem), kickers sans tiret.
- `<title>` : « Ducati Superleggera V4 — 152,2 kg. 224 cv. 500 exemplaires. » + meta description + OG complètes (og:title, og:description, og:type website, og:locale fr_FR, og:site_name) + `theme-color` #000000. Lang « fr ». `aria-label` sur chaque section et sur le lien CTA. `scroll-behavior: auto` (jamais smooth). Aucune manipulation du DOM dans un handler de scroll hors GSAP.
