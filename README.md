# Abyss Injector

Injecteur/patcheur **autonome** pour [Abyss](https://github.com/lemonnv39/abyss-cord)
(fork Vencord/Equicord) — un **.exe portable unique** en **Tauri v2** (Rust +
Svelte 5). Il installe et met à jour Abyss sur **Discord Stable, Canary et PTB**,
et se met à jour **lui-même**. Totalement indépendant (aucun installeur, aucun
autre outil requis).

## Utilisation

1. Télécharge `abyss-injector.exe` depuis la
   [dernière Release](https://github.com/lemonnv39/abyss-injector/releases/latest).
2. Lance-le (portable, rien à installer).
3. Au démarrage il vérifie d'abord **ses propres** mises à jour (notification →
   installe → relance), puis les mises à jour **d'Abyss**.
4. Choisis un Discord (Stable / Canary / PTB) et clique **Installer**. À la
   première install, un autre cord détecté (Vencord/Equicord) est désinstallé
   et Discord remis à neuf automatiquement avant d'installer Abyss.
5. Quand une nouvelle version d'Abyss sort, la ligne affiche **Mettre à jour**.

## Fonctionnement

- **Contenu Abyss** : téléchargé depuis la branche `builds` de
  `lemonnv39/abyss-cord` (`patcher.js` / `preload.js` / `renderer.js` /
  `renderer.css`), détection de MAJ via l'API commits. Voir
  `src-tauri/src/dist_fetch.rs`.
- **Injection** : technique du stub `app.asar` par branche (sauvegarde/restaure
  le vrai `app.asar`), retries anti file-lock, kill/relance de Discord. Voir
  `src-tauri/src/{asar.rs,patcher.rs,discord.rs}`.
- **Self-update** : lit les GitHub Releases de ce repo, télécharge le nouvel
  `abyss-injector.exe` et remplace l'exe en cours (portable, pas de NSIS). Voir
  `src-tauri/src/updater.rs`.

## Développement

Prérequis : [Rust](https://rustup.rs/) stable, Node 22+ (`corepack enable`),
[prérequis Tauri Windows](https://v2.tauri.app/start/prerequisites/) (WebView2 +
Build Tools C++).

```bash
pnpm install
pnpm tauri dev            # lancer en dev
pnpm tauri build --no-bundle   # produire le .exe portable (src-tauri/target/release/abyss-injector.exe)
```

## Publier une version

Aligne la version dans `package.json`, `src-tauri/Cargo.toml` et
`src-tauri/tauri.conf.json`, puis pousse un tag :

```bash
git tag v0.1.0 && git push --tags
```

La CI (`.github/workflows/release.yml`) build le `.exe` portable et publie une
GitHub Release avec l'asset `abyss-injector.exe` — c'est ce que le self-update
des utilisateurs récupère.
