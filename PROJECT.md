# Wakfu Hub — PROJECT.md

Doc de suivi du projet. Je la mets à jour à chaque modif notable pour ne pas avoir à ré-expliquer le projet à chaque prompt. Toi tu peux la modifier pour ajuster les règles/priorités.

## Objectif

Hub perso pour 3 persos Wakfu (Iop, Zobal, Sacrieur) : fiches de sorts à jour, moteur de recherche d'objets par niveau, liens de builds externes, combos et notes de métiers éditables. Inspiré d'outils communautaires (WakForge, Zenith Wakfu) mais recentré sur ces 3 persos avec sa propre DA.

## Architecture technique

- **Fichier unique** : `wakfu-hub.html` — HTML + CSS + JS vanilla, aucun framework, aucun build step. Choix volontaire : doit s'ouvrir en local par double-clic, sans serveur.
- **Données embarquées** en JSON inline dans le `<script>` : `ITEMS` (objets), `SPELLS` (sorts par classe), `ASSETS` (images en base64).
- **Stockage utilisateur** : `localStorage`, clés au format `wakfuhub:{persoKey}:{type}` (`pinned`, `combos`, `notes`, `buildlinks`). Rien n'est envoyé nulle part, tout reste dans le navigateur de l'utilisateur.
- **Icônes d'objets** : embarquées en base64 (WebP 32×32, dédupliquées par `imageId`), stockées dans `ITEM_ICONS = { icons: [...], map: { itemId: iconIndex } }`. Repli sur icône de slot SVG uniquement si un item n'a vraiment aucune icône trouvée à l'extraction.

## Sources de données

| Donnée | Source | Statut |
|---|---|---|
| Sorts Iop / Sacrieur | [Guidactik.com](https://guidactik.com/wakfu/tous-nos-guides-et-astuces-pour-wakfu/) (guides FR, màj fév. 2026) | Complet, retranscrit à la main |
| Sorts Zobal | Guidactik | Partiel — mécanique des masques + résumé des branches seulement. Détail sort par sort pas encore publié par la source à date. |
| Objets, **restreints niveau 50-120** (1987 items) | [WakForge](https://github.com/Tmktahu/WakForge) (MIT) → `item_data.json` + `fr_items.json` (noms FR) | Dépend de la dernière synchro du repo avec Ankama. Dataset complet (7835 items, niv 0-245) disponible si besoin d'élargir. |
| Emblèmes classe, gemmes rareté, icônes élément | [Vertylo/wakassets](https://github.com/Vertylo/wakassets) — embarqués en base64 | OK, usage communautaire non-commercial autorisé par le repo |
| Icônes d'objets (1257 utilisées / 10912 dispo) | wakassets — **embarquées en base64**, compressées WebP 32×32, dédupliquées par `imageId` | OK partout, ~1,6 Mo dans le fichier final |

## Pipeline de régénération

⚠️ Le sandbox Claude ne persiste PAS entre les sessions. Si je dois remodifier le projet, ces étapes sont à refaire :

```bash
# 1. Cloner les sources
git clone --depth 1 https://github.com/Tmktahu/WakForge.git wakforge
git clone --depth 1 https://github.com/Vertylo/wakassets.git wakassets

# 2. Extraire les objets (item_data.json + fr_items.json → items_final.json)
#    garder : id, nom FR, niveau, rareté, slot principal, effets traduits

# 3. Sorts : données écrites à la main dans spells_fr.json
#    (pas de fichier source fiable trouvé — cf. Limitations)

# 4. Assets : convertir en base64 emblèmes (breedsIcons/8,11,14.png),
#    éléments (elements/FIRE,EARTH,AIR,WATER.png), raretés (rarities/0-7.png)

# 5. Injecter items_final.json / spells_fr.json / assets_b64.json
#    dans le template HTML (remplacement de placeholders __ITEMS_JSON__ etc.)
```

## Règles établies (à respecter dans toute future modif)

1. **Aucune donnée inventée.** Si pas de source fiable et à jour trouvée, le dire clairement plutôt que d'halluciner. Wakfu a eu un rework majeur qui rend beaucoup de wikis (fandom, wiki.gg) obsolètes — toujours vérifier la fraîcheur d'une source avant de l'utiliser.
2. **Créditer les sources** dans le footer du site (déjà fait pour Guidactik, WakForge, wakassets, Zenith).
3. **Respecter les licences** : WakForge = MIT ; wakassets = usage communautaire non-commercial explicitement autorisé par Ankama (pas de revente/usage commercial).
4. **Fonctionnel avant esthétique**, mais une passe visuelle a été demandée et appliquée : typo Cinzel/Work Sans, palette violet foncé + or + accent par classe (Iop = braise, Zobal = violet masqué, Sacrieur = rouge sang).
5. **Pas de fausse promesse de synchro.** Aucune API publique Ankama ni Zenith Wakfu n'existe pour lire les données d'un personnage ou d'un build en live. La solution retenue est un stockage manuel de liens (onglet Builds), pas une synchro automatique.
6. **Tout en français.**

## Fichiers importants

- `/mnt/user-data/outputs/wakfu-hub.html` — **le livrable**, fichier unique autonome. C'est la seule chose que tu dois garder/versionner de ton côté.
- `/mnt/user-data/outputs/PROJECT.md` — ce fichier.
- *(éphémères, recréés à chaque session si besoin)* `wakforge/`, `wakassets/`, `items_final.json`, `spells_fr.json`, `assets_b64.json`

## Limitations connues

- **Icônes d'objets** : ne s'affichent que si le fichier `.html` est ouvert directement depuis le disque (double-clic) dans un navigateur normal. Dans l'aperçu publié Claude (et probablement dans l'aperçu du chat), une politique de sécurité bloque le chargement d'images externes → repli sur icône de slot générique. Les emblèmes/gemmes/icônes d'élément, eux, sont embarqués en base64 et s'affichent toujours, partout.
- **Zobal incomplet** : pas de détail sort par sort, seulement la mécanique des masques.
- **Pas de synchro de compte/personnage** : aucune API publique Ankama pour ça.
- **Données objets figées** au moment de l'extraction (pas de mise à jour automatique si Ankama patch le jeu).

## To-do / Roadmap

- [ ] Compléter les sorts détaillés du Zobal dès qu'une source FR à jour et fiable existe
- [ ] Éventuellement ajouter les icônes de stats (PV/PA/PM/Esquive...) sur les effets d'objets dans la liste Stuff
- [ ] Vérifier avec l'utilisateur si les infos Iop/Sacrieur collent bien à ce qu'il voit en jeu
- [ ] Décider si on garde le moteur de recherche d'objets généraliste ou si on le filtre par pertinence de classe
- [ ] Ajouter éventuellement un export/import JSON des données locales (combos/notes/pins) pour ne pas perdre les données si vidage du navigateur

## Changelog

- **v1** — premier jet, anglais, thème sombre générique, sorts issus de wikis obsolètes (contamination Dofus/vieux Wakfu)
- **v2** — passage complet en français + vraies données de sorts à jour (Guidactik) + noms d'objets FR (WakForge)
- **v3** — passe visuelle : typo Cinzel/Work Sans, palette violette/or, emblèmes SVG maison
- **v4** — intégration des vrais assets Ankama via wakassets (emblèmes, gemmes de rareté, icônes élémentaires)
- **v5** — icônes d'objets en lien externe (wakassets) + nouvel onglet Builds (liens Zenith/WakForge sauvegardés manuellement)
- **v6** — correctif majeur : les icônes matchaient sur le mauvais id (`item.id` au lieu du vrai `imageId`) → 0% de couverture réelle. Icônes désormais embarquées en base64 (WebP 32×32, dédupliquées), fonctionnent partout. Dataset "Stuff" restreint aux objets niveau 50-120 sur demande (allège aussi le fichier : 2,15 Mo)
- **v6.1** — clarification (pas de code changé) : le filtre niveau 50-120 et la base d'objets sont **globaux**, partagés à l'identique par les 3 persos (`ITEM_LEVEL_MIN`/`ITEM_LEVEL_MAX`/`ITEMS` sont des constantes uniques, pas liées à `state.char`). Confirmé avec l'utilisateur : même tranche pour Iop, Zobal et Sacrieur.
