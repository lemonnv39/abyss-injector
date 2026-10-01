<script lang="ts">
    // Fond sobre monochrome : deux halos BLANCS diffus (un en haut, un en bas)
    // qui respirent très lentement (transform/opacity — GPU-friendly), sur un
    // noir profond + trame technique + vignette. Discret, jamais envahissant.
    // `dimmed` l'atténue pendant une installation pour focaliser l'UI.
    let { dimmed = false }: { dimmed?: boolean } = $props();
</script>

<div class="backdrop" class:dimmed aria-hidden="true">
    <div class="glow glow-a"></div>
    <div class="glow glow-b"></div>
    <div class="grid"></div>
    <div class="vignette"></div>
</div>

<style>
    .backdrop {
        position: fixed;
        inset: 0;
        z-index: 0;
        overflow: hidden;
        background: radial-gradient(140% 120% at 50% -20%, #141416 0%, var(--bg) 62%);
        transition: opacity var(--duration-slow) var(--ease-out);
    }

    .backdrop.dimmed { opacity: 0.4; }

    .glow {
        position: absolute;
        border-radius: 50%;
        filter: blur(100px);
        will-change: transform;
    }

    .glow-a {
        width: 460px;
        height: 460px;
        top: -200px;
        left: -120px;
        background: radial-gradient(circle, rgba(255, 255, 255, 0.16), transparent 70%);
        animation: drift-a 38s var(--ease-in-out) infinite;
    }

    .glow-b {
        width: 420px;
        height: 420px;
        bottom: -200px;
        right: -100px;
        background: radial-gradient(circle, rgba(255, 255, 255, 0.10), transparent 70%);
        animation: drift-b 46s var(--ease-in-out) infinite;
    }

    /* Trame très discrète, façon "surface technique". */
    .grid {
        position: absolute;
        inset: 0;
        background-image:
            linear-gradient(rgba(255, 255, 255, 0.022) 1px, transparent 1px),
            linear-gradient(90deg, rgba(255, 255, 255, 0.022) 1px, transparent 1px);
        background-size: 34px 34px;
        mask-image: radial-gradient(120% 90% at 50% 30%, #000 40%, transparent 85%);
        -webkit-mask-image: radial-gradient(120% 90% at 50% 30%, #000 40%, transparent 85%);
    }

    .vignette {
        position: absolute;
        inset: 0;
        background: radial-gradient(130% 100% at 50% 40%, transparent 55%, rgba(0, 0, 0, 0.5) 100%);
    }

    @keyframes drift-a {
        0%, 100% { transform: translate(0, 0) scale(1); }
        50% { transform: translate(50px, 36px) scale(1.08); }
    }
    @keyframes drift-b {
        0%, 100% { transform: translate(0, 0) scale(1); }
        50% { transform: translate(-44px, -30px) scale(1.1); }
    }
</style>
