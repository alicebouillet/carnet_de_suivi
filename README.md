# Carnet — déploiement

Cette version fonctionne sans installation ni build : c'est un site statique
(HTML/JS uniquement) que tu héberges gratuitement sur GitHub Pages. Les
données sont stockées dans **Supabase** (base Postgres gratuite) et
synchronisées en temps réel entre les deux téléphones.

## 1. Créer la base de données (Supabase — gratuit)

1. Va sur https://supabase.com et crée un compte (avec GitHub ou email).
2. **New project** → donne-lui un nom (ex. `carnet`), choisis un mot de passe
   pour la base (à conserver, tu n'en auras pas besoin au quotidien) et une
   région proche (ex. `eu-central-1`). Patiente ~2 minutes le temps que le
   projet se crée.
3. Une fois dans le projet, ouvre **SQL Editor** (menu de gauche) → **New query**,
   colle ce script, puis **Run** :

   ```sql
   create table public.carnet (
     id text primary key,
     data jsonb not null default '{}'::jsonb,
     updated_at timestamptz not null default now()
   );

   alter table public.carnet enable row level security;

   create policy "allow read" on public.carnet for select using (true);
   create policy "allow insert" on public.carnet for insert with check (true);
   create policy "allow update" on public.carnet for update using (true);

   insert into public.carnet (id, data) values ('data', '{}'::jsonb);

   alter publication supabase_realtime add table public.carnet;
   ```

   ⚠️ Ces règles sont ouvertes (pas de mot de passe pour lire/écrire) :
   n'importe qui connaissant exactement ton URL et ta clé publique pourrait
   accéder aux données. C'est suffisant pour un usage perso à deux, mais
   évite de partager publiquement ces informations. Dis-moi si tu veux
   qu'on ajoute une vraie authentification plus tard.

4. Va dans **Project Settings** (⚙️) → **API**. Tu y trouves :
   - **Project URL** (ex. `https://xxxxx.supabase.co`)
   - **anon public** key (une longue chaîne de caractères)

## 2. Configurer le fichier

Ouvre `index.html` et remplace ce bloc (cherche `REMPLACE_MOI`) par tes valeurs :

```js
const SUPABASE_URL = "https://xxxxx.supabase.co";
const SUPABASE_ANON_KEY = "eyJhbGciOi...";
```

## 3. Mettre en ligne (GitHub Pages — gratuit)

1. Crée un compte GitHub si besoin (https://github.com), puis un nouveau
   dépôt (**New repository**), nom libre (ex. `carnet`), **Public**.
2. Ajoute les fichiers de ce dossier (`index.html`, `manifest.json`, `sw.js`, `icon.svg`) :
   - soit en les glissant directement dans l'interface GitHub (**Add file → Upload files**),
   - soit via git en ligne de commande si tu es à l'aise avec.
3. Une fois les fichiers poussés : **Settings** (du dépôt) → **Pages** (menu de gauche)
   → Source : **Deploy from a branch** → Branch : `main` / dossier `/ (root)` → **Save**.
4. Après 1-2 minutes, ton site est en ligne à une adresse du type :
   `https://TON-PSEUDO.github.io/carnet/`

## 4. Installer comme une app sur les deux téléphones

Ouvre cette adresse sur chaque téléphone, puis :
- **iPhone (Safari)** : bouton Partager → **Sur l'écran d'accueil**.
- **Android (Chrome)** : menu ⋮ → **Ajouter à l'écran d'accueil** / **Installer l'application**.

L'icône apparaît alors comme une vraie application, en plein écran, sans
barre d'adresse. Les deux téléphones pointent vers le même projet Supabase
donc toute modification faite sur l'un apparaît automatiquement sur l'autre
(le petit point à côté du titre "Carnet" indique l'état de synchronisation).

## Mettre à jour l'app plus tard

Pour ajouter une fonctionnalité, remplace simplement `index.html` dans le
dépôt GitHub (Upload files, en écrasant l'ancien) — la page se met à jour
automatiquement, pas besoin de retoucher Supabase.