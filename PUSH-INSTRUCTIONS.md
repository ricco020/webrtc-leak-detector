# Comment publier ce repo sur GitHub (5 min)

> Préparé par Cortex. Eric exécute quand il veut.

## 1. Créer le repo GitHub
https://github.com/new
- **Name** : `webrtc-leak-detector`
- **Description** : "Single HTML file to detect WebRTC IP leaks in your browser. No dependencies, no tracking."
- **Public**, ne pas cocher README/LICENSE/gitignore (déjà inclus)

## 2. Push
```bash
cd /mnt/c/Users/lerer/Desktop/Affil/webrtc-leak-detector
git remote add origin https://github.com/ricco020/webrtc-leak-detector.git
git branch -M main
git push -u origin main
```

## 3. Polish
- Website : `https://www.anonymflow.com/en/blog/dns-leak-test`
- Topics : `webrtc`, `privacy`, `security`, `vpn`, `ip-leak`, `browser-security`

## 4. Activer GitHub Pages (BONUS recommandé)
Settings → Pages → Branch: main → /(root) → Save
→ Le détecteur devient accessible publiquement sur `https://ricco020.github.io/webrtc-leak-detector/webrtc-detect.html`
→ DR 90 supplémentaire en backlink

## 5. Submissions awesome-lists
Mêmes que dns-leak-detector-cli, ajouter:
- `awesome-privacy`
- `awesome-webrtc` (s'il existe)
