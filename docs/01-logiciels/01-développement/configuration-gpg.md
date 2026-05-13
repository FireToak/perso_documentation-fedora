# Installation et configuration de GPG pour la signature de commits Git

## Informations

- **Auteur :** Louis MEDO
- **Date de création :** 13 mai 2026
- **Date de modification :** 13 mai 2026

---

## Contexte

Ce document détaille la procédure standardisée pour déployer la signature cryptographique des commits Git sur un poste de travail Fedora. L'objectif est de garantir l'intégrité du code, l'authenticité de l'auteur et la conformité avec les bonnes pratiques de sécurité SRE. La procédure inclut la configuration de l'agent GPG pour la gestion du cache de la phrase de passe, optimisant ainsi le flux de travail des développeurs.

---

## Installation de GPG

Sur Fedora, le paquet `gnupg2` est généralement installé par défaut. Il est recommandé de s'assurer que le paquet est présent et à jour.

1. **Vérification et installation des paquets nécessaires.** Mettre à jour la liste des paquets et installer `gnupg2` ainsi que `pinentry-gnome3` (pour l'interface graphique de saisie).

    ```bash
    sudo dnf update -y
    sudo dnf install -y gnupg2 pinentry-gnome3
    ```

---

## Configuration de GPG et Git

Cette section couvre la génération de clés, l'optimisation de l'agent pour le cache de la phrase de passe, et l'intégration avec Git.

1. **Génération d'une nouvelle paire de clés GPG.** Créer une clé avec un algorithme fort (RSA 4096) et une durée de validité définie.

    ```bash
    gpg --full-generate-key
    ```

    *Suivez l'assistant :*

    *   Type de clé : **RSA and RSA**.
    *   Taille : **4096** bits.
    *   Durée de validité : **2y** ou **3y**.
    *   Nom et Email : Doivent correspondre exactement à `git config user.name` et `user.email`.
    *   Passphrase : Choisissez une phrase de passe complexe.

2. **Configuration de l'agent GPG pour le cache (Pinentry).** Configurer le client `pinentry` pour qu'il utilise l'interface graphique de GNOME, permettant une meilleure intégration et la gestion du cache. Cela permet de ne pas devoir retaper le mot de passe de sa signature plusieurs fois par heure.

    ```bash
    # Créer ou éditer le fichier de configuration de l'agent
    nano ~/.gnupg/gpg-agent.conf
    ```

    Ajoutez ou modifiez les lignes suivantes pour définir la durée du cache (en secondes). Ici, le cache est maintenu pendant 1 heure (3600s) avec un maximum de 8 heures (28800s).

    ```text
    # Utilisation de pinentry-gnome3
    pinentry-program /usr/bin/pinentry-gnome3

    # Durée de vie du cache par défaut (1 heure)
    default-cache-ttl 3600

    # Durée de vie maximale du cache (8 heures)
    max-cache-ttl 28800

    # Permettre la saisie via le terminal si nécessaire (optionnel)
    allow-loopback-pinentry
    ```

    *Redémarrez l'agent pour appliquer les changements :*

    ```bash
    gpgconf --kill gpg-agent
    ```

3. **Exportation et sauvegarde des clés.** Sauvegardez la clé privée dans un gestionnaire de mots de passe.

    ```bash
    EMAIL="louis.medo@loutik.fr"

    # Exporter la clé privée (à stocker sécuritairement)
    gpg --armor --export-secret-keys $EMAIL > gpg_private_backup.asc

    # Exporter la clé publique (pour GitHub/GitLab)
    gpg --armor --export $EMAIL > gpg_public.asc
    ```

    **💡 sécurité** : Après avoir copié le contenu de `gpg_private_backup.asc` dans votre gestionnaire de mots de passe, détruisez le fichier temporaire : `shred -u gpg_private_backup.asc`.

4. **Configuration de Git pour signer automatiquement.** Informez Git de l'identité de signature et activez la signature par défaut.

    ```bash
    # Définir l'identité (doit matcher la clé GPG)
    git config user.name "Louis MEDO"
    git config user.email "louis.medo@loutik.fr"

    # Indiquer à Git l'ID de la clé GPG à utiliser
    # Trouvez l'ID avec : gpg --list-secret-keys --keyid-format=long
    KEY_ID="E6B3D814488D0DCB" 
    git config user.signingkey $KEY_ID

    # Activer la signature automatique pour ce dépôt
    git config commit.gpgsign true
    ```

---

## Vérification du fonctionnement de GPG

Validez que la chaîne de signature et le cache de la phrase de passe fonctionnent correctement.

1. **Test de signature et vérification du cache.** Créer un commit de test. La première fois, la fenêtre `pinentry` s'ouvrira pour demander la phrase de passe. Les commits suivants dans la fenêtre de temps définie (1 heure) ne devraient plus la demander.

    ```bash
    echo "Test de signature SRE avec cache" >> test_sre.md
    git add test_sre.md
    git commit -m "test: vérification configuration GPG et agent cache"
    ```

    **Retour attendu :**

    Le processus de commit ne doit retourner aucune erreur. Lors de l'inspection du log, la signature doit apparaître comme valide.

    ```bash
    git log --show-signature -1
    ```

    **Sortie console attendue :**
    ```text
    commit a1b2c3d4...
    gpg: Signature made mer. 13 mai 2026 15:00:00 CEST
    gpg:                using RSA key 79CAB6A684A78A207A5CCB41E6B3D814488D0DCB
    gpg: Good signature from "Louis MEDO <louis.medo@loutik.fr>" [ultimate]
    Author: Louis MEDO <louis.medo@loutik.fr>
    Date:   mer. 13 mai 2026 15:00:00 CEST

        test: vérification configuration GPG et agent cache
    ```

    **Note** : Si vous recommencez un commit immédiatement après et qu'aucune fenêtre de mot de passe n'apparaît, le cache fonctionne correctement.

---

## Ressources

- [Documentation officielle GNU Privacy Guard](https://www.gnupg.org/documentation/)
- [GitHub : Generating a new GPG key](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)
- [Arch Wiki : GnuPG (Section Agent)](https://wiki.archlinux.org/title/GnuPG#gpg-agent)
- [Fedora Security Guide : GPG](https://docs.fedoraproject.org/en-US/quick-docs/gpg-key-management/)