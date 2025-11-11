# Projet-COC

Il s'agit d'un projet [Next.js](https://nextjs.org) intialisé avec [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).
Pour des des raisons de sécurité le code ne peut être partager en public, vous trouverez ici toutes autres informations relatives au site web. <br>

## Prestataire & Sous-traitant
| Prestataire | Rôle | Localisation |
|--------------|------|--------------|
| **Vercel Inc.** | Hébergement du site | UE / USA |
| **Supabase** | Authentification, base de données | UE / USA |
| **Stripe / PayPal** | Traitement des paiements | UE / USA |
| **Resend** | Envoi d’emails transactionnels | UE / USA |

*Ces prestataires appliquent des normes de sécurité et de confidentialité conformes au RGPD (chiffrement, clauses contractuelles types, PCI-DSS, etc.)<br>
Je tiens à préciser que je n'ai aucune action chez eux mais j'apprécie vraiment leurs services open-source/freemium. <br>

### Supabase offre plusieurs avantages par rapport à d'autres solutions. Il utilise PostgreSQL de manière native, sans fork, ce qui garantit une compatibilité complète et un accès à des extensions avancées comme PostGIS, TimescaleDB ou PgVector, souvent disponibles en un seul clic.
De plus, son système de sécurité au niveau des lignes (Row Level Security, RLS) permet de sécuriser les données directement depuis le client, en définissant des politiques d'accès spécifiques, comme "les utilisateurs ne peuvent voir que leurs propres données".
Cela permet une interaction directe avec la base de données depuis le navigateur, protégée par ces règles, sans nécessiter de créer des endpoints API pour chaque opération.<br>

### Vercel est idéal pour l'hébergement du frontend et des fonctions sans serveur, tandis que Supabase fournit une base de données SQL, une authentification sans serveur, un stockage et des fonctions Edge, ce qui permet de couvrir l'ensemble du backend de manière efficace.
L'intégration entre Vercel et Supabase est fluide : elle permet de synchroniser automatiquement les variables d'environnement entre les deux services, de créer des projets Supabase comme ressources Vercel, de gérer les factures via Vercel et de configurer automatiquement les URLs de redirection pour les branches de prévisualisation. <br>

---
# Sécurité Actuelle :
✅ RLS activé (données protégées) <br>
✅ Fonction SQL sécurisée (injection protection) <br>
✅ Validation Zod côté client et serveur (mots de passe forts) <br>
✅ Middleware de protection (routes) <br>
✅ Serveurs situés en région EU (performance + RGPD) <br>
✅ Tokens JWT sécurisés (Supabase) <br>

### Données sensibles bien protégées :
- Les clés secrètes sont uniquement en env serveur.
- Les fichiers sensibles ne sont jamais importés côté client.
- Webhook Stripe tourne en Node runtime.
### Auth correctement implémentée : 
- Auth vérifiée server-side dans (dashboard)/layout.tsx.
- Middleware redirige par défaut (défense en profondeur).
- Les API revalident toujours la session (ne pas faire confiance au middleware seul).
### API routes sécurisées :
- Autorisation par user (session → user.id) sur toutes les mutations/lectures.
- Rate limit sur endpoints sensibles.
- Aucune donnée sensible dans la réponse (pas de service role key).
### Stripe : 
- Escalade d’accès (lecture d’un paiement d’un autre user) → Mitigé par RLS + vérification serveur user_id
- XSS via formulaires/public reviews → Mitigé par validation Zod, escape/encode, composants contrôlés.
### Webhooks Stripe correctement validés :
- Utilisation de stripe.webhooks.constructEvent avec corps brut + STRIPE_WEBHOOK_SECRET.
- Idempotence implémentée (vérif event.id).
- Aucun effet irréversible sans vérif du statut réel du paiement.

# Arborescence 

```text
CertifConfEuro/
├── .next/                              # Next.js 15 (TS/Tailwind)
├── app/
│   ├── api/                            # API Routes Next.js (Backend)
│   │   ├── auth/
│   │   │   ├── callback/
│   │   │   │   └── route.ts            # Callback OAuth/email verification
│   │   │   ├── login/
│   │   │   │   └── route.ts            # Connexion utilisateur
│   │   │   ├── logout/
│   │   │   │   └── route.ts            # Déconnexion utilisateur
│   │   │   └── signup/
│   │   │       └── route.ts            # Inscription utilisateur
│   │   ├── contact/
│   │   │   ├── submit/
│   │   │   │   └── route.ts            # Traitement formulaire de demande COC
│   │   │   └── send-email/
│   │   │       └── route.ts            # Envoi email avec données formulaire
│   │   └── payment/
│   │       ├── create-checkout/
│   │       │   └── route.ts            # Création session paiement Stripe
│   │       └── webhook/
│   │           └── route.ts            # Webhook Stripe (validation paiement)
│   │
│   ├── auth/                           # Pages d'authentification (routes publiques)
│   │   ├── page.tsx                    # Page unique inscription/connexion
│   │   └── layout.tsx
│   ├── dashboard/                      # Zone privée utilisateur (routes protégées)
│   │   ├── page.tsx                    # Page principale Dashboard
│   │   └── layout.tsx                  # Avec protection auth
│   ├── success/
│   │   └── page.tsx                    # Page de confirmation après paiement
│   ├── (legal)/                        # Mentions légales / CGU
│   │   └── ...
│   │
│   ├── components/                     # Tous les composants réutilisables
│   │   ├── home/                       # Accueil
│   │   │   ├── Header.tsx
│   │   │   └── ...etc.tsx
│   │   ├── auth/                       # Authentification
│   │   │   └── ...
│   │   ├── dashboard/                  # Dashboard
│   │   │   └── ...
│   │   └── ui/                         # UI génériques (Logo/Icones)
│   │       └── ...
│   │
│   ├── favicon.ico                     # Logo site
│   ├── globals.css                     # Styles globaux (TailwindCSS)
│   ├── layout.tsx                      # Layout racine de l'application
│   └── page.tsx                        # Page d'accueil
│
├── lib/                                # Utilitaires et configurations
│   ├── supabase/
│   │   ├── client.ts                   # Client Supabase côté navigateur
│   │   └── server.ts                   # Client Supabase côté serveur
│   ├── hooks/
│   │   └── useAutoLogout.tsx           # Déconnexion automatique de l'utilisateur
│   ├── stripe/
│   │   └── server.ts                   # Configuration Stripe serveur
│   ├── resend/
│   │   ├── templates.ts                # Templates d'emails
│   │   └── server.ts                   # Service d'envoi d'emails
│   ├── validations/
│   │   └── auth.ts                     # Validation formulaires auth
│   └── utils.ts                        # Utilitaires généraux
│
├── middleware.ts                       # Middleware protection routes
├── types/                              # Définitions TypeScript
│   ├── auth.ts                         # Types authentification
│   ├── contact.ts                      # Types formulaire COC
│   └── database.ts                     # Types base de données
├── public/
│   └── ...                             # Images
├── .env.local                          # Variables d'environnement (ignoré)
└── ... (fichiers config)
```
---
# Bilan & Conclusion
Cette approche constitue une solution pertinente pour une startup souhaitant démarrer sans s’engager dans des abonnements ou des coûts supplémentaires. Elle présente néanmoins certaines limites. Il restera toutefois possible de migrer le projet vers d’autres plateformes ou modes d’hébergement lorsque la startup aura gagné en maturité. C’est dans cette optique que j’ai veillé à structurer mon code de manière maintenable et évolutive.
