<script lang="ts">
    import { onMount } from "svelte";
    import { getVersion } from "@tauri-apps/api/app";
    import { open, save } from "@tauri-apps/plugin-dialog";
    import { patcherApi } from "../api/patcher";
    import type { FixResult } from "../types";
    import type { InjectorUpdate } from "../api/updater";
    import UpdateBanner from "./UpdateBanner.svelte";

    let {
        pendingUpdate,
        installingUpdate,
        onCheckInjectorUpdate,
        onInstallInjectorUpdate,
    }: {
        pendingUpdate: InjectorUpdate | null;
        installingUpdate: boolean;
        onCheckInjectorUpdate: () => Promise<"found" | "none">;
        onInstallInjectorUpdate: () => void;
    } = $props();

    let appVersion = $state("");

    let importing = $state(false);
    let exporting = $state(false);
    let fixing = $state(false);
    let checkingInjector = $state(false);
    let resultMessage = $state<string | null>(null);
    let resultIsError = $state(false);

    onMount(() => {
        getVersion().then(v => (appVersion = v));
    });

    function report(message: string, isError = false) {
        resultMessage = message;
        resultIsError = isError;
    }

    async function handleCheckInjectorUpdate() {
        resultMessage = null;
        checkingInjector = true;
        try {
            const outcome = await onCheckInjectorUpdate();
            if (outcome === "none") report("Abyss Injector est déjà à jour.");
        } catch (e) {
            report(String(e), true);
        } finally {
            checkingInjector = false;
        }
    }

    async function handleImport() {
        resultMessage = null;
        const path = await open({
            multiple: false,
            filters: [{ name: "Préréglage de plugins Abyss", extensions: ["json"] }],
        });
        if (!path || Array.isArray(path)) return;

        importing = true;
        try {
            await patcherApi.importPluginPresets(path);
            report("Préréglage importé — redémarre Abyss pour l'appliquer.");
        } catch (e) {
            report(String(e), true);
        } finally {
            importing = false;
        }
    }

    async function handleExport() {
        resultMessage = null;
        const path = await save({
            defaultPath: "abyss-plugins.json",
            filters: [{ name: "Préréglage de plugins Abyss", extensions: ["json"] }],
        });
        if (!path) return;

        exporting = true;
        try {
            await patcherApi.exportPluginPresets(path);
            report("Préréglage exporté avec succès.");
        } catch (e) {
            report(String(e), true);
        } finally {
            exporting = false;
        }
    }

    function summarizeFix(results: FixResult[]): string {
        if (results.length === 0) return "Aucune install Abyss détectée à vérifier.";
        const fixed = results.filter(r => r.fixed);
        const failed = results.filter(r => r.was_broken && !r.fixed);
        if (failed.length > 0) {
            return `Échec sur ${failed.map(r => r.branch).join(", ")} : ${failed[0].message ?? "erreur inconnue"}`;
        }
        if (fixed.length > 0) {
            return `Réparé : ${fixed.map(r => r.branch).join(", ")}. Redémarre Discord pour appliquer.`;
        }
        return "Tout est en ordre, aucune réparation nécessaire.";
    }

    async function handleFix() {
        resultMessage = null;
        fixing = true;
        try {
            const results = await patcherApi.fixAbyss();
            const failed = results.some(r => r.was_broken && !r.fixed);
            report(summarizeFix(results), failed);
        } catch (e) {
            report(String(e), true);
        } finally {
            fixing = false;
        }
    }
</script>

<div class="settings">
    <div class="card">
        <div class="card-title">Préréglages de plugins</div>
        <div class="card-subtitle">Sauvegarde ou restaure l'état et les réglages de tes plugins Abyss.</div>
        <div class="card-actions">
            <button class="btn" disabled={importing} onclick={handleImport}>
                {importing ? "Import…" : "Importer un préréglage"}
            </button>
            <button class="btn" disabled={exporting} onclick={handleExport}>
                {exporting ? "Export…" : "Exporter mes préréglages"}
            </button>
        </div>
    </div>

    <div class="card">
        <div class="card-title">Réparation</div>
        <div class="card-subtitle">Vérifie les fichiers injectés et réinstalle automatiquement ce qui est cassé.</div>
        <div class="card-actions">
            <button class="btn btn--primary" disabled={fixing} onclick={handleFix}>
                {fixing ? "Vérification…" : "Fixer Abyss"}
            </button>
        </div>
    </div>

    {#if resultMessage}
        <p class="hint" class:hint--error={resultIsError}>{resultMessage}</p>
    {/if}

    {#if pendingUpdate}
        <UpdateBanner
            version={pendingUpdate.version}
            installing={installingUpdate}
            onInstall={onInstallInjectorUpdate}
        />
    {/if}

    <div class="card">
        <div class="card-title">Abyss Injector</div>
        <div class="card-subtitle">Version {appVersion || "…"}</div>
        <div class="card-actions">
            <button class="btn" disabled={checkingInjector || !!pendingUpdate} onclick={handleCheckInjectorUpdate}>
                {checkingInjector ? "Vérification…" : "Vérifier les mises à jour"}
            </button>
        </div>
        <div class="credit">Powered by Mxxq &amp; Octn dev</div>
    </div>
</div>

<style>
    .settings {
        display: flex;
        flex-direction: column;
        gap: var(--space-3);
        height: 100%;
        padding: 56px 26px 20px;
        box-sizing: border-box;
        overflow-y: auto;
    }

    .card {
        background: var(--surface);
        border: 1px solid var(--border);
        border-radius: var(--radius-md);
        padding: var(--space-4);
        transition: border-color var(--duration-fast) var(--ease-out);
    }
    .card:hover { border-color: var(--border-strong); }

    .card-title {
        font-size: 13.5px;
        font-weight: 650;
        color: var(--text);
    }

    .card-subtitle {
        font-size: 12px;
        color: var(--text-dim);
        margin-top: 3px;
        line-height: 1.45;
    }

    .credit {
        font-size: 11px;
        color: var(--text-faint);
        margin-top: var(--space-3);
    }

    .card-actions {
        display: flex;
        gap: var(--space-2);
        margin-top: var(--space-4);
        flex-wrap: wrap;
    }

    .btn {
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid var(--border-strong);
        color: var(--text);
        padding: 9px 16px;
        border-radius: var(--radius-sm);
        font-size: 12.5px;
        font-weight: 600;
        transition: background var(--duration-fast) var(--ease-out),
            border-color var(--duration-fast) var(--ease-out),
            transform var(--duration-fast) var(--ease-out);
    }
    .btn:hover:not(:disabled) { background: rgba(255, 255, 255, 0.1); border-color: var(--border-strong); }
    .btn:active:not(:disabled) { transform: scale(0.97); }
    .btn:disabled { opacity: 0.5; cursor: default; }

    .btn--primary {
        background: var(--accent);
        border-color: transparent;
        color: var(--on-accent);
        box-shadow: 0 2px 12px rgba(124, 58, 237, 0.3);
    }
    .btn--primary:hover:not(:disabled) { background: var(--accent-hover); }

    .hint {
        margin: 0;
        font-size: 12px;
        color: var(--ok);
        padding: 9px 12px;
        border-radius: var(--radius-sm);
        background: rgba(52, 211, 153, 0.1);
        border: 1px solid rgba(52, 211, 153, 0.3);
    }

    .hint--error {
        color: var(--danger);
        background: rgba(248, 113, 113, 0.1);
        border-color: rgba(248, 113, 113, 0.35);
    }
</style>
