This needs an update but i can't afford it now
# Devia ORM

Une ORM TypeScript légère et moderne pour Expo SQLite, inspirée de Sequelize.

## Caractéristiques

- **API simple et intuitive** - Style Sequelize pour une courbe d'apprentissage facile
- **100% TypeScript** - Typage complet et sûr
- **Décorateurs** - Définissez vos modèles de manière élégante
- **QueryBuilder** - Requêtes SQL générées automatiquement
- **Type-safe** - Autocomplete et vérification de types
- **Léger** - Zéro dépendance (sauf expo-sqlite)
- **Opérateurs avancés** - $gt, $lt, $like, $in, etc.

## Installation

```bash
npm install devia-orm expo-sqlite
```

ou

```bash
yarn add devia-orm expo-sqlite
```

## Configuration

### 1. Activer les décorateurs

Dans votre `tsconfig.json` :

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### 2. Initialiser la base de données

```typescript
import { Database } from "devia-orm";

// Dans votre App.tsx ou point d'entrée
await Database.getInstance().initialize("myapp.db");
```

## Utilisation de base

### Définir un modèle

```typescript
import { Model, Table, Column, PrimaryKey } from "devia-orm";

// Définir l'interface des attributs
interface AchatAttributes {
  id?: number;
  montant: number;
  userId: number;
  description?: string;
  createdAt?: string;
}

// Définir le modèle
@Table("achats")
class Achat extends Model<AchatAttributes> {
  @PrimaryKey()
  id!: number;

  @Column("REAL")
  montant!: number;

  @Column("INTEGER")
  userId!: number;

  @Column("TEXT")
  description?: string;

  @Column("TEXT")
  createdAt?: string;
}
```

### Synchroniser la table

```typescript
// Créer la table si elle n'existe pas
await Achat.sync();

// Ou recréer la table (supprime les données existantes)
await Achat.sync({ force: true });
```

### Créer des enregistrements

```typescript
// Créer un achat
const achat = await Achat.create({
  montant: 199.99,
  userId: 1,
  description: "Achat de livres",
  createdAt: new Date().toISOString(),
});

console.log(achat.id); // ID auto-généré
```

### Lire des enregistrements

```typescript
// Trouver tous les achats
const achats = await Achat.findAll();

// Avec conditions WHERE
const achatsUser = await Achat.findAll({
  where: { userId: 1 },
});

// Avec limite et tri
const derniers = await Achat.findAll({
  limit: 10,
  order: [["createdAt", "DESC"]],
});

// Trouver un seul enregistrement
const achat = await Achat.findOne({
  where: { id: 1 },
});

// Trouver par clé primaire
const achat = await Achat.findByPk(1);
```

### Mettre à jour des enregistrements

```typescript
// Mettre à jour avec WHERE
const updated = await Achat.update({ montant: 299.99 }, { where: { id: 1 } });

console.log(`${updated} lignes mises à jour`);
```

### Supprimer des enregistrements

```typescript
// Supprimer avec WHERE
const deleted = await Achat.destroy({
  where: { userId: 1 },
});

console.log(`${deleted} lignes supprimées`);
```

### Compter des enregistrements

```typescript
const count = await Achat.count({
  where: { userId: 1 },
});
```

## 🔍 Opérateurs avancés

```typescript
// Plus grand que
await Achat.findAll({
  where: {
    montant: { $gt: 100 },
  },
});

// Plus petit que
await Achat.findAll({
  where: {
    montant: { $lt: 500 },
  },
});

// LIKE (recherche de texte)
await Achat.findAll({
  where: {
    description: { $like: "%livre%" },
  },
});

// IN (dans une liste)
await Achat.findAll({
  where: {
    userId: { $in: [1, 2, 3] },
  },
});

// Différent de
await Achat.findAll({
  where: {
    userId: { $ne: 1 },
  },
});

// Combinaison d'opérateurs
await Achat.findAll({
  where: {
    montant: { $gte: 100, $lte: 500 },
    userId: 1,
  },
});
```

## Décorateurs disponibles

### @Table(tableName)

Définit le nom de la table dans la base de données.

```typescript
@Table("mes_achats")
class Achat extends Model<AchatAttributes> {}
```

### @PrimaryKey(autoIncrement?)

Marque une colonne comme clé primaire.

```typescript
@PrimaryKey(true) // auto-increment (défaut)
id!: number;

@PrimaryKey(false) // pas d'auto-increment
uuid!: string;
```

### @Column(type | options)

Définit une colonne.

```typescript
@Column("TEXT")
nom!: string;

@Column({ type: "INTEGER", nullable: false, unique: true })
code!: number;
```

Types disponibles : `"INTEGER"`, `"TEXT"`, `"REAL"`, `"BLOB"`, `"NULL"`

### @AutoIncrement()

Raccourci pour une clé primaire auto-incrémentée.

```typescript
@AutoIncrement()
id!: number;
```

### @NotNull()

Marque une colonne comme non-nullable.

```typescript
@NotNull()
@Column("TEXT")
email!: string;
```

### @Unique()

Marque une colonne comme unique.

```typescript
@Unique()
@Column("TEXT")
email!: string;
```

### @Default(value)

Définit une valeur par défaut.

```typescript
@Default(0)
@Column("INTEGER")
score!: number;
```

## Exemples complets

### Modèle User

```typescript
interface UserAttributes {
  id?: number;
  email: string;
  name: string;
  age?: number;
  isActive?: boolean;
  createdAt?: string;
}

@Table("users")
class User extends Model<UserAttributes> {
  @PrimaryKey()
  id!: number;

  @Unique()
  @NotNull()
  @Column("TEXT")
  email!: string;

  @Column("TEXT")
  name!: string;

  @Column("INTEGER")
  age?: number;

  @Default(1)
  @Column("INTEGER")
  isActive!: boolean;

  @Column("TEXT")
  createdAt?: string;
}

// Utilisation
await User.sync();

const user = await User.create({
  email: "john@example.com",
  name: "John Doe",
  age: 30,
  createdAt: new Date().toISOString(),
});

const adults = await User.findAll({
  where: { age: { $gte: 18 } },
  order: [["name", "ASC"]],
});
```

### Modèle Product

```typescript
interface ProductAttributes {
  id?: number;
  name: string;
  price: number;
  stock: number;
  category: string;
}

@Table("products")
class Product extends Model<ProductAttributes> {
  @AutoIncrement()
  id!: number;

  @NotNull()
  @Column("TEXT")
  name!: string;

  @NotNull()
  @Column("REAL")
  price!: number;

  @Default(0)
  @Column("INTEGER")
  stock!: number;

  @Column("TEXT")
  category!: string;
}

// Utilisation
await Product.sync();

// Recherche multicritères
const productsInStock = await Product.findAll({
  where: {
    stock: { $gt: 0 },
    category: "Electronics",
    price: { $lt: 1000 },
  },
  order: [["price", "ASC"]],
  limit: 20,
});
```

## 🛠️ API Reference

### Model

#### Méthodes statiques

- `findAll(options?)` - Trouver tous les enregistrements
- `findOne(options?)` - Trouver un enregistrement
- `findByPk(id)` - Trouver par clé primaire
- `create(data)` - Créer un enregistrement
- `update(data, options)` - Mettre à jour des enregistrements
- `destroy(options)` - Supprimer des enregistrements
- `count(options?)` - Compter les enregistrements
- `sync(options?)` - Synchroniser la table
- `drop()` - Supprimer la table
- `truncate()` - Vider la table

### FindOptions

```typescript
{
  where?: WhereOptions<T>;
  limit?: number;
  offset?: number;
  order?: [keyof T, "ASC" | "DESC"][];
}
```

### Operators

- `$gt` - Plus grand que
- `$gte` - Plus grand ou égal
- `$lt` - Plus petit que
- `$lte` - Plus petit ou égal
- `$ne` - Différent de
- `$like` - LIKE SQL
- `$in` - IN (liste de valeurs)

## Tests et développement

```bash
# Build
npm run build

# Watch mode
npm run dev

# Type check
npm run typecheck
```

## Notes importantes

1. **Décorateurs** : Assurez-vous d'avoir `experimentalDecorators: true` dans votre tsconfig.json
2. **Initialisation** : Appelez `Database.getInstance().initialize()` avant d'utiliser les modèles
3. **Sync** : Appelez `Model.sync()` pour créer les tables automatiquement
4. **TypeScript** : Définissez toujours une interface pour vos attributs de modèle

## Contribution

Les contributions sont les bienvenues ! N'hésitez pas à ouvrir une issue ou une pull request.

## Licence

MIT

## Remerciements

Inspiré par Sequelize, adapté pour Expo et React Native.
