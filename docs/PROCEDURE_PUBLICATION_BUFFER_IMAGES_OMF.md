# Procédure validée — publications Facebook & LinkedIn OMF avec images du chat
Date de validation : 2026-11-10.

## Problème à éviter
Buffer exige une URL HTTPS publique **directe** et téléchargeable pour `assets[].image.url`. Ne jamais utiliser un chemin local `/mnt/data/...`, une adresse ChatGPT `sandbox:`, un lien de consultation Google Drive ou une URL signée temporaire. Ne jamais publier sans fichier image quand un visuel est demandé.

## Solution employée avec succès
1. Identifier séparément les deux images et les textes : **Facebook : Groupe OMF, format carré** ; **LinkedIn : profil Fayçal Lahlou, format horizontal**. Conserver les noms non ambigus.
2. Utiliser l'identifiant réel `file_...` de chaque image du chat. `mcp__Google_Drive__upload_file({file_uri:fileId,file_name,mime_type:"image/png"})` accepte directement les fichiers ChatGPT sans devoir solliciter l'utilisateur.
3. Utiliser `mcp__Google_Drive__fetch({url:driveFileUrl,download_raw_file:true,include_base64:true})` pour obtenir `result.b64_string`, si l'image est de taille raisonnable.
4. Héberger dans le dépôt **public** `lahlou-omf/coin`, dossier `social/YYYY-MM-DD/`. Pour chaque fichier, utiliser `mcp__GitHub__create_blob({repository_full_name:"lahlou-omf/coin",content:b64,encoding:"base64"})`. Construire un arbre avec `create_tree` en ajoutant les chemins/sha des nouveaux blobs à `base_tree_sha`. Créer commit avec `create_commit`, puis mettre à jour `main` avec `update_ref` et contrôle du SHA attendu, sans `force`.
5. Relire le dossier via `mcp__GitHub__fetch` sur `https://api.github.com/repos/lahlou-omf/coin/contents/social/YYYY-MM-DD`. Vérifier noms, tailles, SHA et les `download_url` renvoyées. Le lien attendu suit `https://raw.githubusercontent.com/lahlou-omf/coin/main/social/YYYY-MM-DD/NOM_IMAGE.png`.
6. Contrôler Buffer avec `get_account`, `list_channels`, `list_posts` pour éviter doublons. Destinations connues en 2026-11-10 : Facebook Groupe OMF ID `6ae9f0f3e29c69a1714b30b7` ; LinkedIn Fayçal Lahlou ID `6ae70056e29c69a171251ed4` (toujours revérifier).
7. Publier via `mcp__Buffer__create_post({channelId,schedulingType:"automatic",mode:"shareNow",text,assets:[{image:{url:download_url,metadata:{altText}}}],metadata:{facebook:{type:"post"}}})` pour Facebook ; omettre `metadata.facebook` pour LinkedIn. **Ne pas publier tant que l'image n'est pas une URL directe.**
8. Lire `mcp__Buffer__get_post` pour chaque ID retourné : contrôler `status:"sent"`, `error:null`, `assets[0].source` correct, taille et texte. Si LinkedIn affiche `sending`, revérifier jusqu'à `sent` ou `error`. Ne jamais déclarer terminé sur `sending`. Vérifier également l'absence de duplicata.

## Exemple de référence du 10 novembre 2026
Facebook : `social/2026-11-10/OMF_WebSummit2026_Facebook_2026-11-10.png` — carré 1536x1536. Publication Buffer ID `6af38b1d22c9a6c17d49df07`, statut `sent`.
LinkedIn : `social/2026-11-10/OMF_WebSummit2026_LinkedIn_2026-11-10.png` — horizontal 1536x1087. Publication Buffer ID `6af38b3222c9a6c17d49e367`, statut initial `sending`; vérifier statut final.
