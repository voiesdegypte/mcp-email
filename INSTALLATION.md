# Brancher vos boîtes mail sur Claude

Vingt minutes, trois boîtes, aucun compte développeur. À la fin, Claude lit, cherche, classe,
prépare des brouillons et envoie sur toutes vos adresses, chacune appelée par son nom court.

Notice pas à pas, écrite pour quelqu'un qui n'a jamais installé de serveur MCP. Le `README.md`
de ce dépôt reste la référence technique.

Ce dépôt est une copie de [tnemelclement/mcp-email](https://github.com/tnemelclement/mcp-email),
le connecteur écrit par Clément Mouly (licence MIT). Le code est le sien, cette notice est l'ajout.

---

## Avant de commencer : quatre vérifications

**1. Votre messagerie.** Gmail, Google Workspace, iCloud, Yahoo, Infomaniak, OVH et la plupart
des serveurs d'entreprise fonctionnent.

**Outlook, Hotmail et Microsoft 365 ne fonctionnent pas.** Microsoft a supprimé la connexion par
mot de passe sur ces protocoles au profit d'un mécanisme que ce serveur ne gère pas. Si vos
boîtes sont sur Microsoft 365, arrêtez ici et demandez-nous l'alternative.

**2. La validation en deux étapes** doit être active sur chaque boîte à brancher. Elle
conditionne la création d'un mot de passe d'application, qui est la clé utilisée ici.

**3. Node.js version 18 ou plus récente** sur la machine. Vérification : `node --version`.

**4. Claude Code ou Claude Desktop** installé et connecté.

## Étape 1 : installer le serveur

```bash
git clone https://github.com/voiesdegypte/mcp-email.git
cd mcp-email
npm install
```

Rien à compiler. Le code est libre (licence MIT) et gratuit.

## Étape 2 : déclarer vos boîtes

```bash
cp accounts.json.example accounts.json
```

Ouvrez `accounts.json`. Un bloc par boîte. La clé du bloc (`perso`, `pro`, `compta`) est le nom
court par lequel vous appellerez cette boîte dans vos demandes à Claude.

```json
{
  "default": "pro",
  "accounts": {
    "pro": {
      "imap_host": "imap.gmail.com",
      "imap_port": 993,
      "smtp_host": "smtp.gmail.com",
      "smtp_port": 587,
      "user": "vous@votresociete.fr",
      "from": "Votre Nom <vous@votresociete.fr>"
    }
  }
}
```

Réglages par fournisseur :

| Fournisseur | Réception (IMAP) | Envoi (SMTP) |
|---|---|---|
| Gmail et Google Workspace | `imap.gmail.com` port 993 | `smtp.gmail.com` port 587 |
| iCloud | `imap.mail.me.com` port 993 | `smtp.mail.me.com` port 587 |
| Yahoo | `imap.mail.yahoo.com` port 993 | `smtp.mail.yahoo.com` port 465 |
| Infomaniak | `mail.infomaniak.com` port 993 | `mail.infomaniak.com` port 587 |
| OVH | `ssl0.ovh.net` port 993 | `ssl0.ovh.net` port 587 |

L'identifiant `user` est toujours l'adresse complète. Exception iCloud : la partie avant le `@`
seulement, c'est la cause d'échec numéro un sur ce fournisseur.

## Étape 3 : créer un mot de passe d'application par boîte

Sur Gmail : https://myaccount.google.com/apppasswords. Google affiche seize caractères avec des
espaces. Collez-les **sans les espaces**.

Puis, pour chaque boîte :

```bash
python3 set-password.py pro
```

La saisie est masquée. Le mot de passe part dans un fichier `.env` lisible par vous seul, jamais
affiché, jamais enregistré dans l'historique du terminal, jamais publié.

N'utilisez jamais le mot de passe de votre compte. Un mot de passe d'application se révoque en
un clic, depuis la même page Google, sans toucher à votre connexion habituelle.

## Étape 4 : brancher à Claude

**Claude Code**, en remplaçant le chemin par celui de votre dossier :

```bash
claude mcp add -s user email -- npx tsx /chemin/absolu/vers/mcp-email/src/index.ts
```

**Claude Desktop**, dans `claude_desktop_config.json` :

```json
{
  "mcpServers": {
    "email": {
      "command": "npx",
      "args": ["tsx", "/chemin/absolu/vers/mcp-email/src/index.ts"]
    }
  }
}
```

Le chemin doit être absolu. Redémarrez Claude Desktop après modification.

### Ajouter une boîte plus tard

Complétez `accounts.json`, lancez `set-password.py` pour la nouvelle clé, puis **redémarrez
Claude**. La liste des comptes est lue au démarrage du connecteur, une boîte ajoutée à chaud
reste invisible jusque-là.

## Étape 5 : vérifier

Demandez à Claude : « liste mes comptes email ». Il doit répondre par vos boîtes et leurs noms
courts. Enchaînez avec « les cinq derniers mails non lus sur pro » pour valider la connexion.

Si la liste apparaît, l'installation est finie.

## Ce que vous pouvez demander ensuite

- « Cherche tous les mails de ce client depuis janvier, sur les trois boîtes »
- « Prépare un brouillon de réponse à ce devis, ton commercial, depuis la boîte pro »
- « Range les newsletters du mois dans le dossier Veille »
- « Télécharge la pièce jointe de la facture reçue hier »

Douze opérations disponibles : lister, chercher, lire, envoyer, répondre dans le fil, préparer un
brouillon, déplacer, étiqueter, marquer lu ou non lu, télécharger une pièce jointe, lister les
dossiers, lister les comptes. La recherche porte sur une boîte à la fois ; pour interroger les
trois, Claude enchaîne trois recherches.

## Sécurité

Le serveur tourne sur votre machine. Vos identifiants restent sur votre disque, dans un fichier
protégé. Vos messages ne passent par aucun intermédiaire autre que votre messagerie et le
fournisseur du modèle.

Claude peut lire **et écrire**, dans des limites précises. **Aucun outil ne supprime un
message** : la suppression n'existe pas dans ce connecteur. Ce qu'il fait, c'est envoyer,
répondre, déplacer un message d'un dossier vers un autre, lui ajouter un libellé et le marquer
lu ou non lu.

**Deux opérations partent sans repasser par vous** : l'envoi direct et la réponse dans le fil.
Le brouillon, lui, se dépose dans vos Brouillons sans rien expédier. Demandez un brouillon pour
tout message qui engage.

Lire un message par Claude le marque comme lu, exactement comme si vous l'aviez ouvert
vous-même. Vous pouvez le repasser en non-lu à la demande.

Commencez par une boîte secondaire, le temps de voir comment il travaille.

Pour couper l'accès : révoquez le mot de passe d'application chez votre fournisseur. L'effet est
immédiat, et votre propre connexion n'est pas affectée.

## Si ça coince

**L'authentification échoue alors que le mot de passe semble bon.** Vérifiez qu'il s'agit du mot
de passe d'application et non de celui du compte, et qu'il a été collé sans espaces.

**Le serveur ne démarre pas.** Le chemin de la configuration doit être absolu, du premier `/`
jusqu'à `src/index.ts`.

**Un dossier reste introuvable.** Les noms varient selon les messageries (`Sent`, `Messages
envoyés`, `[Gmail]/Messages envoyés`). Demandez à Claude de lister les dossiers de la boîte pour
obtenir les noms exacts.

**Gmail refuse la connexion.** Vérifiez que l'accès IMAP est activé dans les réglages Gmail,
onglet « Transfert et POP/IMAP ».
