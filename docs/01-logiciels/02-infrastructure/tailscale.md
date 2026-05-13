# Installation et configuration de Tailscale sur Fedora

## Informations

- **Auteur :** Louis MEDO
- **Date de création :** 5 mai 2026
- **Date de modification :** 5 mai 2026

---

## Contexte

Cette procédure détaille l'installation de Tailscale sur Fedora Workstation en utilisant l'extension GNOME pour bénéficier d'une interface graphique native. Elle inclut également la configuration manuelle via le terminal pour déclarer et activer un nœud de sortie (Exit Node), spécifiquement un routeur OPNsense, afin de sécuriser le trafic sur les réseaux publics.

---

## Installation de Tailscale et de l'extension graphique

1. **Installation du client Tailscale et de l'extension GNOME.**
   Pour intégrer Tailscale directement dans la barre de statut de Fedora (en haut à droite), il est nécessaire d'installer le paquet principal ainsi que le paquet spécifique à l'environnement de bureau GNOME.

    ```bash
    sudo dnf install -y tailscale tailscale-gnome-extension
    ```

    - `tailscale` : Le démon principal qui gère la connexion au réseau maillé (mesh VPN).
    - `tailscale-gnome-extension` : Fournit l'applet graphique permettant de voir l'état de la connexion, d'activer/désactiver le service et de gérer les Exit Nodes sans ligne de commande.

2. **Activation de l'extension et connexion.**
    Une fois l'installation terminée, l'extension doit être activée pour apparaître dans la barre des tâches.

    - Ouvrez l'application « Extensions » (ou allez dans *Paramètres* > *Extensions*).
    - Activez l'interrupteur pour **Tailscale**.
    - Cliquez sur l'icône Tailscale apparaissant dans la barre supérieure, puis cliquez sur **Start** ou **Connect**.
    - Une fenêtre de navigateur s'ouvrira pour vous authentifier via votre fournisseur d'identité (Google, Microsoft, GitHub, etc.). Validez l'accès.

    *Note : L'extension graphique permet de voir les appareils connectés, mais pour des configurations avancées comme la sélection précise d'un Exit Node par son IP, l'usage du terminal reste parfois nécessaire.*

---

## Configuration de l'Exit Node (Routeur OPNsense)

1. **Identification du nœud de sortie.**
Avant de configurer le routage, il faut identifier l'adresse IP Tailscale de votre routeur OPNsense (nommé `rt-loutik` dans cet exemple). Ouvrez un terminal et exécutez :

    ```bash
    tailscale status
    ```

    **Exemple de sortie :**
    ```text
    100.70.105.14  pc-portable-louis-linux  louis.medo@  linux    -
    100.109.32.62  pc-portable-louis        louis.medo@  windows  offline, last seen 23d ago
    100.74.116.72  rt-loutik                louis.medo@  freebsd  active; offers exit node; relay "par", tx 69340 rx 923796
    100.67.12.25   s23-fe-de-louis          louis.medo@  android  -
    ```

    - Repérez la ligne contenant `offers exit node`.
    - Dans cet exemple, le routeur **rt-loutik** a l'adresse IP `100.74.116.72`.

2. **Activation de l'Exit Node via le terminal.**
    Bien que l'extension graphique permette parfois de sélectionner un nœud, la méthode la plus fiable sur Linux consiste à utiliser la commande `set` pour forcer le routage de tout le trafic vers le routeur.

    ```bash
    sudo tailscale set --exit-node=100.74.116.72
    ```

    - `sudo` : Requis car la modification des tables de routage système est une opération privilégiée.
    - `--exit-node=` : Spécifie l'adresse IP (ou le nom de domaine) du peer qui servira de passerelle internet.
    - Remplacez `100.74.116.72` par l'IP réelle de votre routeur si elle diffère.

3. **Vérification du fonctionnement.**
    Pour confirmer que votre trafic web transite bien par le routeur OPNsense et non plus par le Wi-Fi local, vérifiez votre adresse IP publique.

    ```bash
    curl ifconfig.me
    ```

    - Le résultat affiché doit correspondre à l'adresse IP publique de votre connexion domestique (celle du routeur OPNsense), et non celle du réseau Wi-Fi public sur lequel vous êtes connecté physiquement.
    - Dans l'extension GNOME, vous devriez également voir une indication mentionnant que vous utilisez un Exit Node.

---

## Gestion et désactivation

Si vous souhaitez revenir à une connexion directe (désactiver l'Exit Node) tout en restant connecté au réseau Tailscale pour accéder aux autres machines :

```bash
sudo tailscale set --exit-node=
```

- Laisser la valeur vide après le signe `=` réinitialise le routage par défaut et désactive l'usage du nœud de sortie.

---

## Ressources

- [Documentation officielle Tailscale - Exit Nodes](https://tailscale.com/docs/features/exit-nodes)
- [Documentation Tailscale sur Linux](https://tailscale.com/docs/install/linux)
- [Forum OPNsense - Tailscale Plugin](https://forum.opnsense.org/index.php?topic=45530.0)