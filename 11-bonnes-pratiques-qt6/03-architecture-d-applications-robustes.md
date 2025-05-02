# 11.3 Architecture d'applications robustes

## Introduction à l'architecture robuste

La création d'une application Qt6 durable et évolutive nécessite plus que de bonnes pratiques de codage : elle demande une architecture solide. Dans ce chapitre, nous allons explorer comment concevoir des applications robustes qui peuvent résister à l'évolution des exigences, faciliter la maintenance et garantir la fiabilité.

## Principes fondamentaux d'une architecture robuste

### Séparation des préoccupations

L'un des principes les plus importants consiste à diviser votre application en composants distincts, chacun responsable d'un aspect spécifique :

```cpp
// Au lieu d'une classe qui fait tout
class ApplicationMonolithique {
public:
    void chargerDonnees();
    void enregistrerDonnees();
    void afficherInterface();
    void gererAuthentification();
    // ... et bien plus
};

// Préférer des classes spécialisées
class GestionnaireDonnees {
public:
    void charger();
    void enregistrer();
};

class GestionnaireAuthentification {
public:
    bool authentifier(const QString& utilisateur, const QString& motDePasse);
};

class InterfaceUtilisateur {
public:
    void afficher();
};
```

### Principes SOLID

Les principes SOLID constituent une base solide pour concevoir des applications robustes :

1. **S** - Principe de responsabilité unique (Single Responsibility)
2. **O** - Principe d'ouverture/fermeture (Open/Closed)
3. **L** - Principe de substitution de Liskov (Liskov Substitution)
4. **I** - Principe de ségrégation des interfaces (Interface Segregation)
5. **D** - Principe d'inversion des dépendances (Dependency Inversion)

Voyons un exemple du principe de responsabilité unique :

```cpp
// MAUVAIS: Classe avec plusieurs responsabilités
class Utilisateur {
public:
    void enregistrerEnBase();
    void afficherDansInterface();
    void envoyerEmail();
};

// BON: Classes avec responsabilités uniques
class ModeleUtilisateur {
public:
    void enregistrerEnBase();
};

class VueUtilisateur {
public:
    void afficher();
};

class ServiceEmail {
public:
    void envoyerEmail(const QString& destinataire, const QString& sujet);
};
```

## Modèles d'architecture pour Qt6

### Model-View-Controller (MVC)

Le modèle MVC divise l'application en trois composants principaux :

1. **Modèle** : Gestion des données et logique métier
2. **Vue** : Présentation des données à l'utilisateur
3. **Contrôleur** : Gestion des interactions utilisateur

Qt6 implémente une variante de ce modèle avec son architecture Model/View :

```cpp
// Exemple de modèle
class ListeUtilisateursModel : public QAbstractListModel {
private:
    QList<Utilisateur> m_utilisateurs;

public:
    int rowCount(const QModelIndex& parent = QModelIndex()) const override {
        return m_utilisateurs.size();
    }

    QVariant data(const QModelIndex& index, int role = Qt::DisplayRole) const override {
        if (!index.isValid())
            return QVariant();

        if (role == Qt::DisplayRole)
            return m_utilisateurs[index.row()].nom();

        return QVariant();
    }
};

// Utilisation du modèle avec une vue
void configureInterface() {
    ListeUtilisateursModel* modele = new ListeUtilisateursModel(this);
    QListView* vue = new QListView(this);
    vue->setModel(modele);
}
```

### Model-View-ViewModel (MVVM)

Le modèle MVVM étend le MVC en ajoutant une couche ViewModel qui s'occupe de la logique de présentation :

```cpp
// ViewModel (adaptateur entre le modèle et la vue)
class UtilisateursViewModel : public QObject {
    Q_OBJECT
    Q_PROPERTY(QString texteRecherche READ texteRecherche WRITE setTexteRecherche NOTIFY texteRechercheChanged)

private:
    QString m_texteRecherche;
    ListeUtilisateursModel* m_modele;
    QSortFilterProxyModel* m_modeleFiltré;

public:
    UtilisateursViewModel(QObject* parent = nullptr) : QObject(parent) {
        m_modele = new ListeUtilisateursModel(this);
        m_modeleFiltré = new QSortFilterProxyModel(this);
        m_modeleFiltré->setSourceModel(m_modele);
    }

    QString texteRecherche() const {
        return m_texteRecherche;
    }

    void setTexteRecherche(const QString& texte) {
        if (m_texteRecherche != texte) {
            m_texteRecherche = texte;
            m_modeleFiltré->setFilterFixedString(texte);
            emit texteRechercheChanged();
        }
    }

    QAbstractItemModel* modeleFiltre() const {
        return m_modeleFiltré;
    }

signals:
    void texteRechercheChanged();
};
```

### Modèle de composition

Le modèle de composition favorise la composition d'objets plutôt que l'héritage pour construire des fonctionnalités complexes :

```cpp
// Au lieu d'utiliser une hiérarchie d'héritage complexe
class DocumentTexte : public Document {
    // ...
};

class DocumentImage : public Document {
    // ...
};

class DocumentMixte : public DocumentTexte, public DocumentImage {
    // Problèmes potentiels d'héritage multiple !
};

// Utiliser la composition
class Document {
private:
    QList<ComponentDocument*> m_composants;

public:
    void ajouterComposant(ComponentDocument* composant) {
        m_composants.append(composant);
    }

    void enregistrer() {
        for (ComponentDocument* composant : m_composants) {
            composant->enregistrer();
        }
    }
};

class ComposantTexte : public ComponentDocument {
    // ...
};

class ComposantImage : public ComponentDocument {
    // ...
};
```

## Organisation du code dans les projets Qt6

### Structure de projet recommandée

Une bonne organisation de projet améliore la lisibilité et la maintenance :

```
MonProjet/
├── src/
│   ├── main.cpp
│   ├── modeles/
│   │   ├── utilisateur_modele.h
│   │   └── utilisateur_modele.cpp
│   ├── vues/
│   │   ├── fenetre_principale.h
│   │   └── fenetre_principale.cpp
│   └── services/
│       ├── authentification_service.h
│       └── authentification_service.cpp
├── ressources/
│   ├── images/
│   └── styles/
├── tests/
│   ├── test_utilisateur_modele.h
│   └── test_utilisateur_modele.cpp
└── MonProjet.pro
```

### Utiliser des espaces de noms

Les espaces de noms aident à organiser le code et éviter les conflits :

```cpp
// Dans utilisateur_modele.h
namespace MonProjet {
namespace Modeles {

class Utilisateur {
    // ...
};

} // namespace Modeles
} // namespace MonProjet

// Utilisation
void fonction() {
    MonProjet::Modeles::Utilisateur utilisateur;
}

// Ou avec using
using namespace MonProjet::Modeles;
void fonction() {
    Utilisateur utilisateur;
}
```

## Gestion des dépendances

### Injection de dépendances

L'injection de dépendances permet de découpler les composants :

```cpp
// Mauvaise approche : dépendance directe
class ServiceUtilisateur {
private:
    BaseDeDonnees m_baseDeDonnees;  // Dépendance directe et rigide

public:
    void enregistrerUtilisateur(const Utilisateur& utilisateur) {
        m_baseDeDonnees.inserer("utilisateurs", utilisateur.toMap());
    }
};

// Bonne approche : injection de dépendances
class IBaseDeDonnees {
public:
    virtual ~IBaseDeDonnees() = default;
    virtual void inserer(const QString& table, const QMap<QString, QVariant>& donnees) = 0;
};

class ServiceUtilisateur {
private:
    IBaseDeDonnees* m_baseDeDonnees;  // Interface abstraite

public:
    // Injection par constructeur
    ServiceUtilisateur(IBaseDeDonnees* baseDeDonnees) : m_baseDeDonnees(baseDeDonnees) {}

    void enregistrerUtilisateur(const Utilisateur& utilisateur) {
        m_baseDeDonnees->inserer("utilisateurs", utilisateur.toMap());
    }
};
```

### Fabrique (Factory)

Le pattern Factory simplifie la création d'objets complexes :

```cpp
class IDocument {
public:
    virtual ~IDocument() = default;
    virtual void ouvrir() = 0;
    virtual void enregistrer() = 0;
};

class DocumentTexte : public IDocument {
    // Implémentation...
};

class DocumentImage : public IDocument {
    // Implémentation...
};

// Fabrique de documents
class FabriqueDocument {
public:
    enum TypeDocument { Texte, Image };

    static IDocument* creerDocument(TypeDocument type) {
        switch (type) {
            case Texte:
                return new DocumentTexte();
            case Image:
                return new DocumentImage();
            default:
                return nullptr;
        }
    }
};

// Utilisation
void fonction() {
    IDocument* document = FabriqueDocument::creerDocument(FabriqueDocument::Texte);
    document->ouvrir();
}
```

## Gestion des configurations

### Externaliser les configurations

Rendez votre application configurable sans recompilation :

```cpp
// Utiliser QSettings pour les configurations
void configurerApplication() {
    QSettings settings("MaSociete", "MonApplication");

    // Lire des paramètres
    QString serveur = settings.value("serveur", "localhost").toString();
    int port = settings.value("port", 8080).toInt();

    // Utiliser les paramètres
    connecterAuServeur(serveur, port);
}

// Permettre à l'utilisateur de modifier les paramètres
void sauvegarderConfiguration(const QString& serveur, int port) {
    QSettings settings("MaSociete", "MonApplication");
    settings.setValue("serveur", serveur);
    settings.setValue("port", port);
}
```

### Structure de configuration hiérarchique

Pour des configurations plus complexes :

```cpp
// Configuration hiérarchique avec QSettings
void configurerApplication() {
    QSettings settings("MaSociete", "MonApplication");

    settings.beginGroup("reseau");
    QString serveur = settings.value("serveur", "localhost").toString();
    int port = settings.value("port", 8080).toInt();
    settings.endGroup();

    settings.beginGroup("interface");
    bool modeNuit = settings.value("modeNuit", false).toBool();
    QString langue = settings.value("langue", "fr").toString();
    settings.endGroup();
}
```

## Gestion des erreurs

### Utiliser les exceptions de manière appropriée

```cpp
// Définir des exceptions personnalisées
class ErreurBaseDeDonnees : public std::exception {
private:
    QString m_message;

public:
    ErreurBaseDeDonnees(const QString& message) : m_message(message) {}

    const char* what() const noexcept override {
        return m_message.toLocal8Bit().constData();
    }
};

// Utiliser les exceptions
void fonctionQuiPeutEchouer() {
    try {
        // Code qui peut échouer
        if (!connexionReussie) {
            throw ErreurBaseDeDonnees("Impossible de se connecter à la base de données");
        }
    } catch (const ErreurBaseDeDonnees& e) {
        // Journaliser l'erreur
        qCritical() << "Erreur de base de données:" << e.what();

        // Informer l'utilisateur
        QMessageBox::critical(nullptr, "Erreur", e.what());
    }
}
```

### Système de journalisation robuste

```cpp
// Configurer un système de journalisation
void configurerJournalisation() {
    // Rediriger les messages vers un fichier
    static QFile fichierLog("application.log");
    if (fichierLog.open(QIODevice::WriteOnly | QIODevice::Append)) {
        qInstallMessageHandler([](QtMsgType type, const QMessageLogContext& context, const QString& msg) {
            static QTextStream flux(&fichierLog);

            // Obtenir l'horodatage
            QString horodatage = QDateTime::currentDateTime().toString("yyyy-MM-dd hh:mm:ss.zzz");

            // Formater le message
            QString message = QString("[%1] %2: %3 (%4:%5)")
                .arg(horodatage)
                .arg(typeMsgEnTexte(type))
                .arg(msg)
                .arg(context.file)
                .arg(context.line);

            // Écrire dans le fichier
            flux << message << Qt::endl;
        });
    }
}

// Utilisation du système de journalisation
void exemple() {
    qDebug() << "Information de débogage";
    qInfo() << "Information générale";
    qWarning() << "Avertissement";
    qCritical() << "Erreur critique";
}
```

## Modèles de conception utiles en Qt6

### Singleton

Le pattern Singleton garantit qu'une classe n'a qu'une seule instance :

```cpp
class GestionnaireConfiguration {
public:
    // Obtenir l'instance unique
    static GestionnaireConfiguration& instance() {
        static GestionnaireConfiguration instance;
        return instance;
    }

    // Empêcher la copie
    GestionnaireConfiguration(const GestionnaireConfiguration&) = delete;
    void operator=(const GestionnaireConfiguration&) = delete;

    // Méthodes de l'instance
    QString obtenirParametre(const QString& cle) {
        return m_parametres.value(cle).toString();
    }

private:
    // Constructeur privé
    GestionnaireConfiguration() {
        chargerParametres();
    }

    void chargerParametres() {
        // Charger depuis un fichier ou une base de données
    }

    QMap<QString, QVariant> m_parametres;
};

// Utilisation
void fonction() {
    QString serveur = GestionnaireConfiguration::instance().obtenirParametre("serveur");
}
```

### Observateur (implémenté via les signaux et slots)

Qt implémente le pattern Observateur avec son système de signaux et slots :

```cpp
class SujetObservable : public QObject {
    Q_OBJECT

private:
    int m_valeur;

public:
    SujetObservable(QObject* parent = nullptr) : QObject(parent), m_valeur(0) {}

    int valeur() const {
        return m_valeur;
    }

    void setValeur(int valeur) {
        if (m_valeur != valeur) {
            m_valeur = valeur;
            emit valeurChangee(valeur);
        }
    }

signals:
    void valeurChangee(int nouvellValeur);
};

class Observateur : public QObject {
    Q_OBJECT

public:
    Observateur(QObject* parent = nullptr) : QObject(parent) {}

public slots:
    void surValeurChangee(int nouvelleValeur) {
        qDebug() << "La valeur a changé:" << nouvelleValeur;
    }
};

// Utilisation
void fonction() {
    SujetObservable* sujet = new SujetObservable();
    Observateur* observateur = new Observateur();

    // Connecter l'observateur au sujet
    connect(sujet, &SujetObservable::valeurChangee,
            observateur, &Observateur::surValeurChangee);

    // Modifier la valeur - déclenche le signal
    sujet->setValeur(42);
}
```

## Exemple complet d'architecture

Voici un exemple simplifié d'une architecture robuste pour une application de gestion de tâches :

```cpp
// --- INTERFACES ---

// Interface pour la persistance des données
class IDepotTaches {
public:
    virtual ~IDepotTaches() = default;
    virtual QList<Tache> obtenirTaches() = 0;
    virtual void ajouterTache(const Tache& tache) = 0;
    virtual void supprimerTache(int id) = 0;
};

// --- IMPLÉMENTATIONS ---

// Implémentation concrète avec SQLite
class DepotTachesSQLite : public IDepotTaches {
private:
    QSqlDatabase m_db;

public:
    DepotTachesSQLite() {
        m_db = QSqlDatabase::addDatabase("QSQLITE");
        m_db.setDatabaseName("taches.db");
        if (m_db.open()) {
            initialiserBaseDeDonnees();
        } else {
            throw ErreurBaseDeDonnees("Impossible d'ouvrir la base de données");
        }
    }

    QList<Tache> obtenirTaches() override {
        // Implémentation...
    }

    void ajouterTache(const Tache& tache) override {
        // Implémentation...
    }

    void supprimerTache(int id) override {
        // Implémentation...
    }

private:
    void initialiserBaseDeDonnees() {
        // Créer les tables si nécessaire
    }
};

// --- SERVICES ---

// Service de gestion des tâches
class ServiceTaches {
private:
    IDepotTaches* m_depot;

public:
    ServiceTaches(IDepotTaches* depot) : m_depot(depot) {}

    QList<Tache> obtenirTachesNonCompletes() {
        QList<Tache> toutes = m_depot->obtenirTaches();
        QList<Tache> nonCompletes;

        for (const Tache& tache : toutes) {
            if (!tache.estCompletee()) {
                nonCompletes.append(tache);
            }
        }

        return nonCompletes;
    }

    void ajouterTache(const QString& titre, const QString& description) {
        Tache nouvelleTache(0, titre, description, false);
        m_depot->ajouterTache(nouvelleTache);
    }
};

// --- CONTRÔLEUR ---

// Contrôleur pour l'interface utilisateur
class ControleurTaches : public QObject {
    Q_OBJECT

private:
    ServiceTaches* m_service;

public:
    ControleurTaches(ServiceTaches* service, QObject* parent = nullptr)
        : QObject(parent), m_service(service) {}

public slots:
    void surAjouterTacheRequis(const QString& titre, const QString& description) {
        try {
            m_service->ajouterTache(titre, description);
            emit tacheAjouteeAvecSucces();
        } catch (const std::exception& e) {
            emit erreurSurvenue(QString("Erreur lors de l'ajout: %1").arg(e.what()));
        }
    }

signals:
    void tacheAjouteeAvecSucces();
    void erreurSurvenue(const QString& message);
};

// --- MODÈLE ---

// Modèle pour les vues
class ModeleTaches : public QAbstractListModel {
    Q_OBJECT

private:
    QList<Tache> m_taches;
    ServiceTaches* m_service;

public:
    ModeleTaches(ServiceTaches* service, QObject* parent = nullptr)
        : QAbstractListModel(parent), m_service(service) {
        rafraichirTaches();
    }

    int rowCount(const QModelIndex& parent = QModelIndex()) const override {
        return m_taches.size();
    }

    QVariant data(const QModelIndex& index, int role = Qt::DisplayRole) const override {
        // Implémentation...
    }

public slots:
    void rafraichirTaches() {
        beginResetModel();
        m_taches = m_service->obtenirTachesNonCompletes();
        endResetModel();
    }
};

// --- POINT D'ENTRÉE ---

int main(int argc, char* argv[]) {
    QApplication app(argc, argv);

    try {
        // Configuration de la journalisation
        configurerJournalisation();

        // Créer les composants
        IDepotTaches* depot = new DepotTachesSQLite();
        ServiceTaches* service = new ServiceTaches(depot);
        ControleurTaches* controleur = new ControleurTaches(service);
        ModeleTaches* modele = new ModeleTaches(service);

        // Configurer l'interface
        FenetrePrincipale fenetre(controleur, modele);
        fenetre.show();

        // Connecter les signaux de mise à jour
        QObject::connect(controleur, &ControleurTaches::tacheAjouteeAvecSucces,
                         modele, &ModeleTaches::rafraichirTaches);

        return app.exec();
    } catch (const std::exception& e) {
        QMessageBox::critical(nullptr, "Erreur critique",
                             QString("L'application n'a pas pu démarrer: %1").arg(e.what()));
        return 1;
    }
}
```

## Conclusions et bonnes pratiques

Pour créer des applications Qt6 robustes, souvenez-vous de ces principes essentiels :

1. **Séparation des préoccupations** : Divisez votre application en composants distincts.
2. **Interfaces abstraites** : Utilisez des interfaces pour découpler les implémentations.
3. **Injection de dépendances** : Injectez les dépendances plutôt que de les créer en interne.
4. **Gestion d'erreurs** : Prévoyez et gérez les erreurs de manière robuste.
5. **Tests unitaires** : Concevez votre architecture pour faciliter les tests.
6. **Évolutivité** : Créez une architecture qui peut évoluer avec les besoins.
7. **Documentation** : Documentez vos décisions architecturales et les interactions entre composants.

En suivant ces principes, vous créerez des applications Qt6 qui pourront évoluer, être maintenues facilement et rester robustes face aux changements.

Dans la prochaine section, nous explorerons les meilleures pratiques pour sécuriser vos applications Qt6.
