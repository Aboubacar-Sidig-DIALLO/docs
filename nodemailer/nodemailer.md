# Nodemailer

### Table des matières

1. Introduction
2. Configuration de base
3. Transport SMTP
4. Defaults (valeurs par défaut)
5. Envoi d'emails
6. Headers personnalisés
7. Tags et métadonnées
8. Exemples pratiques

### Introduction

Nodemailer est un module Node.js pour envoyer des emails. Il supporte SMTP, Sendmail, Amazon SES, et d'autres transports.

```javascript
npm install nodemailer
npm install @types/nodemailer  # Pour TypeScript
```

#### concept de base

```
const transporter = nodemailer.createTransport({
  // Configuration du transport
});

await transporter.sendMail({
  // Options de l'email
});
```

### Configuration de base

#### Structure minimale

```
import * as nodemailer from 'nodemailer';

const transporter = nodemailer.createTransport({
  host: 'smtp.example.com',
  port: 587,
  secure: false, // true pour 465, false pour autres ports
  auth: {
    user: 'username',
    pass: 'password',
  },
});
```

### Transport SMTP

#### Configuration SMTP standard

**1. SMTP avec STARTTLS (port 587)**

```
transport: {
  host: 'smtp.gmail.com',
  port: 587,
  secure: false, // STARTTLS
  auth: {
    user: 'votre-email@gmail.com',
    pass: 'votre-mot-de-passe-app', // Mot de passe d'application
  },
}
```

**2. SMTP avec SSL (port 465)**

```
transport: {
  host: 'smtp.gmail.com',
  port: 465,
  secure: true, // SSL direct
  auth: {
    user: 'votre-email@gmail.com',
    pass: 'votre-mot-de-passe-app',
  },
}
```

**3. SMTP avec OAuth2**

```
transport: {
  host: 'smtp.gmail.com',
  port: 587,
  secure: false,
  auth: {
    type: 'OAuth2',
    user: 'votre-email@gmail.com',
    clientId: 'votre-client-id',
    clientSecret: 'votre-client-secret',
    refreshToken: 'votre-refresh-token',
    accessToken: 'votre-access-token', // Optionnel
  },
}
```

**4. Mandrill (Mailchimp Transactional)**

```
transport: {
  host: 'smtp.mandrillapp.com',
  port: 587,
  secure: false, // STARTTLS
  auth: {
    user: 'votre-username-mandrill',
    pass: 'votre-api-key-mandrill',
  },
}
```

**5. SendGrid**

```
transport: {
host: 'smtp.sendgrid.net',
port: 587,
secure: false,
auth: {
user: 'apikey', // Toujours "apikey"
pass: 'votre-api-key-sendgrid',
},
}
```

#### Options avancées du transport

```
transport: {
  host: 'smtp.example.com',
  port: 587,
  secure: false,
  auth: {
    user: 'username',
    pass: 'password',
  },
  
  // Options de connexion
  connectionTimeout: 2000,      // Timeout de connexion (ms)
  greetingTimeout: 2000,         // Timeout de salutation (ms)
  socketTimeout: 2000,           // Timeout socket (ms)
  
  // Pool de connexions
  pool: true,                    // Utiliser un pool de connexions
  maxConnections: 5,             // Nombre max de connexions
  maxMessages: 100,              // Nombre max de messages par connexion
  
  // Rate limiting
  rateDelta: 1000,               // Intervalle entre messages (ms)
  rateLimit: 5,                  // Nombre max de messages par intervalle
  
  // Debug
  debug: true,                   // Afficher les logs de debug
  logger: true,                  // Logger les événements
  
  // TLS/SSL
  tls: {
    rejectUnauthorized: false,   // Accepter les certificats auto-signés
    ciphers: 'SSLv3',            // Ciphers spécifiques
  },
}
```

#### Transports alternatifs

**1. Sendmail (serveur local)**

```
transport: {
  sendmail: true,
  newline: 'unix', // ou 'windows'
  path: '/usr/sbin/sendmail', // Chemin vers sendmail
}
```

**2. Stream Transport (pour tests)**

```
transport: {
  streamTransport: true,
  newline: 'unix',
  buffer: true,
}
```

### Defaults

Les defaults définissent des valeurs par défaut appliquées à tous les emails.

#### Configuration de base

```
defaults: {
  from: '"Nom de l'expéditeur" <email@example.com>',
  replyTo: 'reply@example.com',
  subject: 'Sujet par défaut',
}
```

#### Format de l'expéditeur

```
// Format simple
from: 'email@example.com'

// Format avec nom
from: '"Nom de l'expéditeur" <email@example.com>'

// Format avec nom et email séparés
from: {
  name: 'Nom de l'expéditeur',
  address: 'email@example.com',
}
```

#### Exemple complet

```
defaults: {
  from: '"Elyamaje Académie" <noreply@elyamaje.com>',
  replyTo: 'support@elyamaje.com',
  subject: 'Message depuis Elyamaje Académie',
  // Ces valeurs seront utilisées si non spécifiées dans sendMail()
}
```

#### Priorité des valeurs

1. Valeurs explicites dans sendMail()
2. Valeurs dans defaults
3. Valeurs système (si aucune n'est définie)

### Envoi d'emails

#### Méthode sendMail()

```
const result = await transporter.sendMail({
  // Destinataires
  to: 'recipient@example.com',
  cc: 'cc@example.com',
  bcc: 'bcc@example.com',
  
  // Expéditeur (surcharge les defaults)
  from: 'sender@example.com',
  replyTo: 'reply@example.com',
  
  // Contenu
  subject: 'Sujet de l\'email',
  text: 'Version texte de l\'email',
  html: '<h1>Version HTML de l\'email</h1>',
  
  // Pièces jointes
  attachments: [
    {
      filename: 'document.pdf',
      path: './documents/document.pdf',
    },
  ],
  
  // Headers personnalisés
  headers: {
    'X-Custom-Header': 'valeur',
  },
});
```

#### Format des destinataires

```
// Format simple
to: 'email@example.com'

// Format avec nom
to: '"John Doe" <john@example.com>'

// Format objet
to: {
  name: 'John Doe',
  address: 'john@example.com',
}

// Tableau de destinataires
to: [
  'email1@example.com',
  '"John Doe" <john@example.com>',
  { name: 'Jane', address: 'jane@example.com' },
]
```

#### Contenu de l'email

**HTML uniquement**

```
await transporter.sendMail({
  to: 'user@example.com',
  subject: 'Email HTML',
  html: '<h1>Bonjour</h1><p>Ceci est un email HTML.</p>',
});
```

**Texte uniquement**

```
await transporter.sendMail({
  to: 'user@example.com',
  subject: 'Email texte',
  text: 'Bonjour, ceci est un email texte.',
});
```

**HTML et texte (recommandé)**

```
await transporter.sendMail({
  to: 'user@example.com',
  subject: 'Email complet',
  text: 'Bonjour, ceci est la version texte.',
  html: '<h1>Bonjour</h1><p>Ceci est la version HTML.</p>',
});
```

#### Pièces jointes

```
attachments: [
  // Fichier depuis le système de fichiers
  {
    filename: 'document.pdf',
    path: './documents/document.pdf',
  },
  
  // Fichier depuis URL
  {
    filename: 'image.png',
    path: 'https://example.com/image.png',
  },
  
  // Fichier depuis buffer
  {
    filename: 'data.txt',
    content: Buffer.from('Contenu du fichier'),
  },
  
  // Fichier en base64
  {
    filename: 'image.jpg',
    content: 'base64-encoded-string',
    encoding: 'base64',
  },
  
  // Fichier avec cid (pour images inline)
  {
    filename: 'logo.png',
    path: './logo.png',
    cid: 'logo@example.com', // Utiliser dans HTML: <img src="cid:logo@example.com">
  },
]
```

### Headers personnalisés

#### Headers standards

```
headers: {
  // Headers standards
  'From': 'sender@example.com',
  'To': 'recipient@example.com',
  'Subject': 'Sujet',
  'Reply-To': 'reply@example.com',
  'Message-ID': '<unique-id@example.com>',
  'Date': new Date().toUTCString(),
  
  // Headers de priorité
  'Priority': 'high',
  'Importance': 'high',
  'X-Priority': '1', // 1=High, 3=Normal, 5=Low
  
  // Headers de catégorisation
  'X-Mailer': 'MyApp/1.0',
  'X-Category': 'transactional',
}
```

#### Headers Mandrill spécifiques

```
headers: {
  // Tags pour analytics
  'X-MC-Tags': 'welcome,user-initiated,high-priority',
  
  // Métadonnées structurées (JSON)
  'X-MC-Metadata': JSON.stringify({
    userId: '123',
    campaignId: 'campaign-456',
    source: 'api',
  }),
  
  // Configuration du tracking
  'X-MC-Track': 'opens,clicks', // ou 'opens', 'clicks', ou 'opens,clicks'
  
  // Auto-génération de contenu
  'X-MC-AutoText': 'true',  // Génère automatiquement la version texte
  'X-MC-AutoHTML': 'true',  // Génère automatiquement la version HTML
  
  // URLs de tracking personnalisées
  'X-MC-TrackingDomain': 'track.example.com',
  
  // Subaccount
  'X-MC-Subaccount': 'subaccount-id',
  
  // Google Analytics
  'X-MC-GoogleAnalytics': 'campaign-name,source,medium,term,content',
  
  // Preserve les recipients
  'X-MC-PreserveRecipients': 'true',
}
```

#### Exemple d'utilisation

```
await transporter.sendMail({
  to: 'user@example.com',
  subject: 'Bienvenue',
  html: '<h1>Bienvenue !</h1>',
  headers: {
    // Tags pour analytics Mandrill
    'X-MC-Tags': 'welcome,onboarding,user-initiated',
    
    // Métadonnées structurées
    'X-MC-Metadata': JSON.stringify({
      mailType: 'WELCOME',
      userId: '12345',
      sentAt: new Date().toISOString(),
    }),
    
    // Tracking
    'X-MC-Track': 'opens,clicks',
    
    // Header personnalisé
    'X-Custom-Reference': 'welcome-email-12345',
  },
});
```

### Tags et métadonnées

#### Tags (via X-MC-Tags)

Les tags servent à catégoriser les emails pour l'analytics et le filtrage.

**Format**

```
// Tags séparés par des virgules
'X-MC-Tags': 'tag1,tag2,tag3'

// Exemple
'X-MC-Tags': 'welcome,user-initiated,high-priority'
```

**Bonnes pratiques**

* Utiliser kebab-case : password-reset, welcome-email
* Maximum 50 caractères par tag
* Maximum 10 tags par email
* Éviter les espaces et caractères spéciaux

**Exemples de tags**

```
// Par type d'email
'welcome', 'password-reset', 'invoice', 'newsletter'

// Par source
'user-initiated', 'system-generated', 'admin-sent'

// Par priorité
'urgent', 'high-priority', 'normal', 'low-priority'

// Par campagne
'campaign-2024-q1', 'promo-summer', 'black-friday'

// Par version
'v2-template', 'new-design', 'a-b-test-variant-a'
```

#### Métadonnées (via X-MC-Metadata)

Les métadonnées stockent des données structurées en JSON.

**Format**

```
'X-MC-Metadata': JSON.stringify({
  key1: 'value1',
  key2: 'value2',
  nested: {
    data: 'value',
  },
})
```

**Exemple complet**

```
headers: {
  'X-MC-Metadata': JSON.stringify({
    // Type d'email
    mailType: 'WELCOME',
    
    // Identifiants
    userId: '12345',
    orderId: 'order-789',
    campaignId: 'campaign-456',
    
    // Timestamps
    sentAt: new Date().toISOString(),
    expiresAt: '2024-12-31T23:59:59Z',
    
    // Source
    source: 'api',
    version: 'v2',
    
    // Données personnalisées
    customData: {
      subscriptionTier: 'premium',
      language: 'fr',
    },
  }),
}
```

#### Utilisation dans le code

```
// Préparer les tags
const tags: string[] = [];
tags.push('welcome');
tags.push('user-initiated');
if (isUrgent) {
  tags.push('urgent');
}

// Préparer les métadonnées
const metadata = {
  mailType: 'WELCOME',
  userId: user.id,
  sentAt: new Date().toISOString(),
};

// Ajouter aux headers
const headers: Record<string, string> = {};

if (tags.length > 0) {
  headers['X-MC-Tags'] = tags.join(',');
}

headers['X-MC-Metadata'] = JSON.stringify(metadata);

// Envoyer l'email
await transporter.sendMail({
  to: user.email,
  subject: 'Bienvenue',
  html: '<h1>Bienvenue !</h1>',
  headers,
});
```

### Exemples pratiques

#### Exemple 1 : Email de bienvenue

```
await transporter.sendMail({
  to: user.email,
  subject: 'Bienvenue sur notre plateforme',
  html: `
    <h1>Bienvenue ${user.name} !</h1>
    <p>Merci de vous être inscrit.</p>
  `,
  headers: {
    'X-MC-Tags': 'welcome,onboarding,user-initiated',
    'X-MC-Metadata': JSON.stringify({
      mailType: 'WELCOME',
      userId: user.id,
      sentAt: new Date().toISOString(),
    }),
    'X-MC-Track': 'opens,clicks',
  },
});
```

#### Exemple 2 : Réinitialisation de mot de passe

```
const resetToken = generateResetToken();
const resetLink = `${baseUrl}/reset-password?token=${resetToken}`;

await transporter.sendMail({
  to: user.email,
  subject: 'Réinitialisation de votre mot de passe',
  html: `
    <h1>Réinitialisation de mot de passe</h1>
    <p>Cliquez sur le lien pour réinitialiser votre mot de passe :</p>
    <a href="${resetLink}">Réinitialiser</a>
  `,
  headers: {
    'X-MC-Tags': 'password-reset,security,urgent',
    'X-MC-Metadata': JSON.stringify({
      mailType: 'RESET_PASSWORD',
      userId: user.id,
      tokenId: resetToken,
      expiresAt: new Date(Date.now() + 3600000).toISOString(), // 1h
    }),
    'X-MC-Track': 'clicks', // Important pour tracker les clics
  },
});
```

#### Exemple 3 : Newsletter

```
await transporter.sendMail({
  to: subscribers.map(s => s.email),
  subject: 'Newsletter - Décembre 2024',
  html: newsletterHtml,
  headers: {
    'X-MC-Tags': 'newsletter,monthly,december-2024',
    'X-MC-Metadata': JSON.stringify({
      mailType: 'NEWSLETTER',
      campaignId: 'newsletter-dec-2024',
      sentAt: new Date().toISOString(),
    }),
    'X-MC-Track': 'opens,clicks',
    'X-MC-GoogleAnalytics': 'newsletter,email,monthly,december-2024',
  },
});
```

#### Exemple 4 : Configuration complète avec NestJS

```
// mail.module.ts
@Module({
  imports: [
    MailerModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        transport: {
          host: configService.get<string>('MAIL_HOST'),
          port: configService.get<number>('MAIL_PORT', 587),
          secure: configService.get<boolean>('MAIL_SECURE', false),
          auth: {
            user: configService.get<string>('MAIL_USER'),
            pass: configService.get<string>('MAIL_PASSWORD'),
          },
          debug: process.env.NODE_ENV === 'development',
          logger: process.env.NODE_ENV === 'development',
        },
        defaults: {
          from: `"${configService.get<string>('MAIL_FROM_NAME')}" <${configService.get<string>('MAIL_FROM_ADDRESS')}>`,
          replyTo: configService.get<string>('MAIL_REPLY_TO'),
        },
      }),
      inject: [ConfigService],
    }),
  ],
})
export class MailModule {}
```

```
// mail.service.ts
@Injectable()
export class MailService {
  constructor(private readonly mailerService: MailerService) {}

  async sendWelcomeEmail(user: User) {
    const tags = ['welcome', 'onboarding', 'user-initiated'];
    const metadata = {
      mailType: 'WELCOME',
      userId: user.id,
      sentAt: new Date().toISOString(),
    };

    return this.mailerService.sendMail({
      to: user.email,
      subject: 'Bienvenue !',
      html: this.generateWelcomeTemplate(user),
      headers: {
        'X-MC-Tags': tags.join(','),
        'X-MC-Metadata': JSON.stringify(metadata),
        'X-MC-Track': 'opens,clicks',
      },
    });
  }
}
```

### Résumé des bonnes pratiques

#### Configuration

* Utiliser des variables d'environnement pour les credentials
* Activer le debug en développement
* Configurer les timeouts appropriés
* Utiliser un pool de connexions pour la production

#### Envoi

* Toujours fournir HTML et texte
* Utiliser des tags pour l'analytics
* Structurer les métadonnées en JSON
* Activer le tracking (opens, clicks)
* Valider les emails avant envoi

#### Tags

* Utiliser kebab-case
* Maximum 50 caractères par tag
* Maximum 10 tags par email
* Catégoriser par type, source, priorité

#### Métadonnées

* Utiliser JSON.stringify()
* Inclure des identifiants (userId, orderId)
* Ajouter des timestamps
* Structurer les données de manière cohérente
