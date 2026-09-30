<script lang="ts">
    // Fond sobre et léger : quelques halos radiaux flous qui dérivent très
    // lentement (transform/opacity seulement — GPU-friendly), sur un dark de
    // base + une vignette. Pas de vidéo, pas de canvas, pas de « trou noir ».
    // `dimmed` (pendant une installation) atténue le tout pour focaliser l'UI.
    let { dimmed = false }: { dimmed?: boolean } = $props();
</script>

<div class="backdrop" class:dimmed aria-hidden="true">
    <div class="glow glow-a"></div>
    <div class="glow glow-b"></div>
    <div class="glow glow-c"></div>
    <div class="vignette"></div>
</div>

<style>
    .backdrop {
        position: fixed;
        inset: 0;
        z-index: 0;
        overflow: hidden;
        background:
            radial-gradient(120% 120% at 50% -10%, #0c0b16 0%, var(--bg) 55%);
        transition: opacity var(--duration-slow) var(--ease-out);
    }

    .backdrop.dimmed {
        opacity: 0.35;
    }

    .glow {
        position: absolute;
        border-radius: 50%;
        filter: blur(90px);
        opacity: 0.5;
        will-change: transform;
    }

    /* Violet Abyss, en haut à gauche — la source lumineuse principale. */
    .glow-a {
        width: 520px;
        height: 520px;
        top: -180px;
        left: -140px;
        background: radial-gradient(circle, rgba(139, 92, 246, 0.55), transparent 70%);
        animation: drift-a 34s var(--ease-in-out) infinite;
    }

    /* Indigo profond, en bas à droite. */
    .glow-b {
        width: 460px;
        height: 460px;
        bottom: -160px;
        right: -120px;
        background: radial-gradient(circle, rgba(79, 70, 229, 0.5), transparent 70%);
        animation: drift-b 42s var(--ease-in-out) infinite;
    }

    /* Petite lueur froide, discrète, au centre-bas — juste une respiration. */
    .glow-c {
        width: 360px;
        height: 360px;
        bottom: -80px;
        left: 40%;
        background: radial-gradient(circle, rgba(45, 212, 191, 0.14), transparent 70%);
        animation: drift-c 50s var(--ease-in-out) infinite;
    }

    .vignette {
        position: absolute;
        inset: 0;
        background: radial-gradient(130% 100% at 50% 40%, transparent 55%, rgba(0, 0, 0, 0.55) 100%);
        pointer-events: none;
    }

    @keyframes drift-a {
        0%, 100% { transform: translate(0, 0) scale(1); }
        50% { transform: translate(60px, 40px) scale(1.08); }
    }
    @keyframes drift-b {
        0%, 100% { transform: translate(0, 0) scale(1); }
        50% { transform: translate(-50px, -34px) scale(1.1); }
    }
    @keyframes drift-c {
        0%, 100% { transform: translate(0, 0) scale(1); opacity: 0.14; }
        50% { transform: translate(-40px, 20px) scale(1.14); opacity: 0.22; }
    }
</style>
