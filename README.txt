# 📦 StockManager CI — Guide de démarrage

Application de gestion de stock — INP-HB IC-GL 2025-2026  
Architecture : **JavaFX 21 + JDBC + MySQL 8 + Apache POI**

---

## ⚙️ Prérequis

| Outil | Version minimum |
|---|---|
| JDK | 17 ou 21 |
| Maven | 3.8+ |
| MySQL | 8.0+ |
| Scene Builder | 21 (optionnel, pour éditer les FXML) |

---

## 🗄️ Étape 1 — Créer la base de données

Ouvrez un terminal MySQL ou un outil comme **DBeaver / MySQL Workbench** et exécutez :

```sql
-- Le fichier complet est dans :
src/main/resources/database.sql
```

Ce script crée la base `stockmanager_ci` avec les 5 tables et insère les données initiales.

**Comptes par défaut :**

| Login | Mot de passe | Rôle |
|---|---|---|
| `admin` | `admin123` | ADMIN |
| `kouame` | `kouame2026` | GESTIONNAIRE |

---

## 🔧 Étape 2 — Configurer la connexion JDBC

Ouvrez le fichier :
```
src/main/java/com/inphb/icgl/stocks/utils/DatabaseConnection.java
```

Modifiez les constantes selon votre configuration MySQL :
```java
private static final String URL      = "jdbc:mysql://localhost:3306/stockmanager_ci?...";
private static final String USER     = "root";        // ← votre user MySQL
private static final String PASSWORD = "";            // ← votre mot de passe MySQL
```

---

## ▶️ Étape 3 — Lancer l'application

```bash
# Depuis la racine du projet (là où se trouve pom.xml)
mvn clean javafx:run
```

---

## 📦 Étape 4 — Générer le JAR exécutable

```bash
mvn clean package
# Le JAR se trouve dans : target/stockmanager-ci-1.0.jar
java -jar target/stockmanager-ci-1.0.jar
```

---

## 🗂️ Structure du projet

```
stockmanager-ci/
├── pom.xml
└── src/main/
    ├── java/com/inphb/icgl/stocks/
    │   ├── MainApp.java            ← Point d'entrée
    │   ├── SplashScreen.java       ← Écran de démarrage (3s)
    │   ├── model/                  ← POJO avec JavaFX Properties
    │   ├── repository/             ← Interfaces DAO
    │   ├── dao/                    ← Implémentations SQL
    │   ├── controller/             ← Contrôleurs FXML
    │   └── utils/                  ← DB, Session, Export, Password
    └── resources/
        ├── fxml/                   ← Vues (éditables dans Scene Builder)
        ├── css/styles.css
        └── database.sql            ← Script de création de la BD
```

---

## 🔑 Fonctionnalités implémentées

- [x] Splash Screen animé (3 secondes, StageStyle.UNDECORATED)
- [x] Authentification avec hashage SHA-256
- [x] Blocage après 3 tentatives (30 secondes)
- [x] Tableau de bord avec 4 indicateurs KPI
- [x] CRUD Catégories avec contrainte d'intégrité
- [x] CRUD Fournisseurs avec contrainte d'intégrité
- [x] CRUD Produits avec alertes visuelles (rouge si stock ≤ minimum)
- [x] Mouvements de stock (ENTREE/SORTIE) avec transaction atomique
- [x] Vérification stock insuffisant avant sortie
- [x] Historique immuable des mouvements (pas de modification/suppression)
- [x] Export Excel XLSX avec Apache POI (lignes d'alerte colorées en rouge)
- [x] Module Utilisateurs réservé aux ADMIN
- [x] Pagination LIMIT/OFFSET sur tous les TableView (15 lignes/page)
- [x] Recherche en temps réel (filtrage dynamique)
- [x] SessionManager pour l'utilisateur connecté
- [x] Menu latéral avec masquage conditionnel (GESTIONNAIRE ne voit pas Utilisateurs)

---

## 💡 Points techniques importants

### Sécurité mots de passe
Les mots de passe sont hashés en **SHA-256** avant stockage.  
La classe `PasswordUtil.sha256(String)` est utilisée systématiquement.

### Transaction atomique (Mouvements)
La création d'un mouvement et la mise à jour du stock se font dans **une seule transaction** avec `conn.setAutoCommit(false)`. En cas d'erreur, un `rollback()` est effectué.

### Pagination SQL
```sql
-- Page N avec 15 lignes par page :
SELECT * FROM produits ORDER BY designation LIMIT 15 OFFSET (N-1)*15
```

### Alerte stock visuelle
```java
// Dans ProduitController.java
tableProduits.setRowFactory(tv -> new TableRow<Produit>() {
    protected void updateItem(Produit p, boolean empty) {
        super.updateItem(p, empty);
        if (p != null && !empty && p.isEnAlerte())
            setStyle("-fx-background-color: #FFE0E0;");
        else setStyle("");
    }
});
```
