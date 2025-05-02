# 4.5 Stockage local avec QSettings

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

## Introduction à QSettings

QSettings est une classe simple et puissante fournie par Qt pour stocker et récupérer les paramètres d'une application de manière persistante. C'est la solution idéale pour sauvegarder les préférences utilisateur, l'état de l'interface et d'autres configurations légères.

Contrairement aux autres méthodes de persistance que nous avons vues (SQL, QDataStream, JSON, XML), QSettings est spécifiquement conçue pour le stockage de paires clé/valeur de manière simple et efficace, sans nécessiter de format complexe.

## Avantages de QSettings

QSettings présente plusieurs avantages :

- **Simplicité d'utilisation** : API très intuitive (get/set)
- **Multiplateforme** : S'adapte automatiquement au système d'exploitation
- **Pas de format à gérer** : Le format de stockage est géré par Qt
- **Performances** : Optimisé pour les accès fréquents à de petites quantités de données
- **Intégration native** : Utilise le registre sous Windows, les fichiers .plist sous macOS, etc.

## Fonctionnement selon les plateformes

QSettings s'adapte automatiquement à la plateforme utilisée :

| Plateforme | Emplacement de stockage | Format |
|------------|-------------------------|--------|
| Windows | Registre Windows | Clés de registre |
| macOS | ~/Library/Preferences | Fichiers .plist |
| Linux/Unix | ~/.config | Fichiers INI |

Cette adaptation est transparente pour le développeur : le même code fonctionnera sur toutes les plateformes sans modification.

## Utilisation de base de QSettings

Voici comment utiliser QSettings pour stocker et récupérer des paramètres simples :

```cpp
#include <QSettings>
#include <QDebug>

void exempleQSettings()
{
    // Création d'un objet QSettings avec les informations d'organisation et d'application
    QSettings settings("MaSociete", "MonApplication");

    // Écriture de valeurs
    settings.setValue("general/langue", "fr");
    settings.setValue("general/modeNuit", true);
    settings.setValue("editeur/taillePolice", 12);
    settings.setValue("editeur/famillePolice", "Courier New");

    // Configuration d'un tableau
    settings.beginWriteArray("fichiers_recents");
    for (int i = 0; i < 5; ++i) {
        settings.setArrayIndex(i);
        settings.setValue("chemin", QString("/chemin/vers/fichier%1.txt").arg(i));
        settings.setValue("date_ouverture", QDateTime::currentDateTime());
    }
    settings.endArray();

    // Lecture de valeurs simples
    QString langue = settings.value("general/langue", "en").toString();
    bool modeNuit = settings.value("general/modeNuit", false).toBool();
    int taillePolice = settings.value("editeur/taillePolice", 10).toInt();

    qDebug() << "Langue:" << langue;
    qDebug() << "Mode nuit:" << modeNuit;
    qDebug() << "Taille de police:" << taillePolice;

    // Lecture d'un tableau
    int nbFichiersRecents = settings.beginReadArray("fichiers_recents");
    for (int i = 0; i < nbFichiersRecents; ++i) {
        settings.setArrayIndex(i);
        QString chemin = settings.value("chemin").toString();
        QDateTime date = settings.value("date_ouverture").toDateTime();

        qDebug() << "Fichier récent" << i << ":" << chemin
                 << "(ouvert le" << date.toString("dd/MM/yyyy hh:mm") << ")";
    }
    settings.endArray();

    // Vérifier si une clé existe
    if (settings.contains("general/theme")) {
        qDebug() << "La clé 'theme' existe";
    } else {
        qDebug() << "La clé 'theme' n'existe pas";
    }

    // Obtenir la liste des clés dans un groupe
    settings.beginGroup("general");
    QStringList cles = settings.childKeys();
    qDebug() << "Clés dans le groupe 'general':" << cles;
    settings.endGroup();
}
```

## Initialisation de QSettings

Vous pouvez initialiser QSettings de plusieurs façons :

```cpp
// Méthode 1 : Spécifier l'organisation et l'application
QSettings settings("MaSociete", "MonApplication");

// Méthode 2 : Utiliser les valeurs par défaut de l'application
// (QCoreApplication::organizationName() et QCoreApplication::applicationName())
QSettings settings;

// Méthode 3 : Spécifier le format et le scope explicitement
QSettings settings(QSettings::IniFormat, QSettings::UserScope,
                   "MaSociete", "MonApplication");

// Méthode 4 : Utiliser un fichier INI spécifique
QSettings settings("/chemin/vers/mon/fichier.ini", QSettings::IniFormat);
```

## Organisation des paramètres en groupes

QSettings permet d'organiser les paramètres en groupes hiérarchiques pour une meilleure lisibilité :

```cpp
void exempleGroupes()
{
    QSettings settings("MaSociete", "MonApplication");

    // Utilisation de groupes
    settings.beginGroup("interface");
    settings.setValue("theme", "clair");
    settings.setValue("langue", "fr");

    // Sous-groupe
    settings.beginGroup("fenetre_principale");
    settings.setValue("largeur", 800);
    settings.setValue("hauteur", 600);
    settings.setValue("position_x", 100);
    settings.setValue("position_y", 100);
    settings.setValue("maximisee", false);
    settings.endGroup(); // Fin du sous-groupe fenetre_principale

    settings.endGroup(); // Fin du groupe interface

    // Accès direct avec des chemins
    int largeur = settings.value("interface/fenetre_principale/largeur").toInt();
    qDebug() << "Largeur de la fenêtre :" << largeur;

    // Supprimer un groupe entier et toutes ses sous-clés
    settings.remove("interface/fenetre_principale");
}
```

## Stockage de types complexes

QSettings peut stocker tous les types de base de Qt ainsi que certains types complexes :

```cpp
void exemplesTypesComplexes()
{
    QSettings settings("MaSociete", "MonApplication");

    // Types de base
    settings.setValue("entier", 42);
    settings.setValue("reel", 3.14159);
    settings.setValue("texte", "Bonjour le monde");
    settings.setValue("booleen", true);

    // Types plus complexes
    settings.setValue("date", QDate::currentDate());
    settings.setValue("heure", QTime::currentTime());
    settings.setValue("dateHeure", QDateTime::currentDateTime());

    // Liste de chaînes
    QStringList langues = {"fr", "en", "es", "de", "it"};
    settings.setValue("langues_disponibles", langues);

    // Couleurs
    QColor couleurPrimaire(0, 120, 215);
    QColor couleurSecondaire(255, 185, 0);
    settings.setValue("couleur_primaire", couleurPrimaire);
    settings.setValue("couleur_secondaire", couleurSecondaire);

    // Lecture
    QStringList languesLues = settings.value("langues_disponibles").toStringList();
    QColor couleurLue = settings.value("couleur_primaire").value<QColor>();

    qDebug() << "Langues disponibles :" << languesLues.join(", ");
    qDebug() << "Couleur primaire RGB :" << couleurLue.red() << couleurLue.green() << couleurLue.blue();
}
```

Pour stocker des types personnalisés, vous devrez les convertir en types supportés par QSettings :

```cpp
// Classe personnalisée
class Profil {
public:
    Profil(const QString &nom = "", int age = 0) : m_nom(nom), m_age(age) {}

    QString nom() const { return m_nom; }
    int age() const { return m_age; }

    // Conversion vers QVariantMap pour stockage
    QVariantMap toVariantMap() const {
        QVariantMap map;
        map["nom"] = m_nom;
        map["age"] = m_age;
        return map;
    }

    // Création depuis QVariantMap
    static Profil fromVariantMap(const QVariantMap &map) {
        return Profil(map.value("nom").toString(),
                     map.value("age").toInt());
    }

private:
    QString m_nom;
    int m_age;
};

// Utilisation avec QSettings
void stockerProfil() {
    QSettings settings;

    Profil profil("Alice Dupont", 28);

    // Stockage
    settings.setValue("profil_utilisateur", profil.toVariantMap());

    // Récupération
    QVariantMap map = settings.value("profil_utilisateur").toMap();
    Profil profilCharge = Profil::fromVariantMap(map);

    qDebug() << "Profil chargé :" << profilCharge.nom() << profilCharge.age();
}
```

## Synchronisation et portée des paramètres

QSettings stocke parfois les paramètres en mémoire cache pour améliorer les performances. Pour s'assurer que les modifications sont écrites sur le disque, utilisez la méthode `sync()` :

```cpp
void synchroniserParametres() {
    QSettings settings;

    // Modification de plusieurs paramètres
    settings.setValue("param1", "valeur1");
    settings.setValue("param2", "valeur2");

    // Force l'écriture sur le disque
    settings.sync();

    // Vérification des erreurs
    if (settings.status() != QSettings::NoError) {
        qDebug() << "Erreur lors de la synchronisation des paramètres";
    }
}
```

La portée (scope) des paramètres détermine s'ils sont spécifiques à l'utilisateur ou à tous les utilisateurs du système :

```cpp
// Paramètres spécifiques à l'utilisateur (par défaut)
QSettings userSettings(QSettings::UserScope, "MaSociete", "MonApplication");

// Paramètres pour tous les utilisateurs (nécessite des privilèges administrateur)
QSettings systemSettings(QSettings::SystemScope, "MaSociete", "MonApplication");
```

## Classe de gestion des préférences

Voici un exemple de classe qui encapsule la gestion des préférences d'une application :

```cpp
// preferences.h
#ifndef PREFERENCES_H
#define PREFERENCES_H

#include <QObject>
#include <QColor>
#include <QFont>
#include <QStringList>

class Preferences : public QObject
{
    Q_OBJECT

public:
    // Singleton pattern
    static Preferences& instance();

    // Langue
    QString langue() const;
    void setLangue(const QString &langue);
    QStringList languesDisponibles() const;

    // Interface
    bool modeNuit() const;
    void setModeNuit(bool actif);

    QColor couleurPrimaire() const;
    void setCouleurPrimaire(const QColor &couleur);

    // Éditeur
    QFont policeEditeur() const;
    void setPoliceEditeur(const QFont &police);

    int tailleTabulation() const;
    void setTailleTabulation(int taille);

    bool surlignageSyntaxe() const;
    void setSurlignageSyntaxe(bool actif);

    // Fichiers récents
    QStringList fichiersRecents() const;
    void ajouterFichierRecent(const QString &chemin);
    void effacerFichiersRecents();

    // Sessions
    bool restaurerSession() const;
    void setRestaurerSession(bool restaurer);

    // Sauvegarde des paramètres
    void charger();
    void sauvegarder();
    void restaurerDefauts();

signals:
    void langueModifiee(const QString &langue);
    void modeNuitModifie(bool actif);
    void couleurPrimaireModifiee(const QColor &couleur);
    void policeEditeurModifiee(const QFont &police);
    void tailleTabulationModifiee(int taille);
    void surlignageSyntaxeModifie(bool actif);
    void fichiersRecentsModifies();
    void restaurerSessionModifie(bool restaurer);

private:
    // Constructeur privé (singleton)
    explicit Preferences(QObject *parent = nullptr);
    ~Preferences();

    // Empêcher la copie
    Preferences(const Preferences&) = delete;
    Preferences& operator=(const Preferences&) = delete;

    // Paramètres avec valeurs par défaut
    QString m_langue = "fr";
    bool m_modeNuit = false;
    QColor m_couleurPrimaire = QColor(0, 120, 215);
    QFont m_policeEditeur = QFont("Consolas", 11);
    int m_tailleTabulation = 4;
    bool m_surlignageSyntaxe = true;
    QStringList m_fichiersRecents;
    bool m_restaurerSession = true;

    static const int MAX_FICHIERS_RECENTS = 10;
};

#endif // PREFERENCES_H
```

```cpp
// preferences.cpp
#include "preferences.h"
#include <QSettings>
#include <QApplication>
#include <QDebug>

Preferences& Preferences::instance()
{
    static Preferences instance;
    return instance;
}

Preferences::Preferences(QObject *parent) : QObject(parent)
{
    charger();
}

Preferences::~Preferences()
{
    sauvegarder();
}

QString Preferences::langue() const
{
    return m_langue;
}

void Preferences::setLangue(const QString &langue)
{
    if (m_langue != langue) {
        m_langue = langue;
        emit langueModifiee(langue);
    }
}

QStringList Preferences::languesDisponibles() const
{
    return {"fr", "en", "es", "de", "it"};
}

bool Preferences::modeNuit() const
{
    return m_modeNuit;
}

void Preferences::setModeNuit(bool actif)
{
    if (m_modeNuit != actif) {
        m_modeNuit = actif;
        emit modeNuitModifie(actif);
    }
}

QColor Preferences::couleurPrimaire() const
{
    return m_couleurPrimaire;
}

void Preferences::setCouleurPrimaire(const QColor &couleur)
{
    if (m_couleurPrimaire != couleur) {
        m_couleurPrimaire = couleur;
        emit couleurPrimaireModifiee(couleur);
    }
}

QFont Preferences::policeEditeur() const
{
    return m_policeEditeur;
}

void Preferences::setPoliceEditeur(const QFont &police)
{
    if (m_policeEditeur != police) {
        m_policeEditeur = police;
        emit policeEditeurModifiee(police);
    }
}

int Preferences::tailleTabulation() const
{
    return m_tailleTabulation;
}

void Preferences::setTailleTabulation(int taille)
{
    if (m_tailleTabulation != taille) {
        m_tailleTabulation = taille;
        emit tailleTabulationModifiee(taille);
    }
}

bool Preferences::surlignageSyntaxe() const
{
    return m_surlignageSyntaxe;
}

void Preferences::setSurlignageSyntaxe(bool actif)
{
    if (m_surlignageSyntaxe != actif) {
        m_surlignageSyntaxe = actif;
        emit surlignageSyntaxeModifie(actif);
    }
}

QStringList Preferences::fichiersRecents() const
{
    return m_fichiersRecents;
}

void Preferences::ajouterFichierRecent(const QString &chemin)
{
    // Éviter les doublons
    m_fichiersRecents.removeAll(chemin);

    // Ajouter au début de la liste
    m_fichiersRecents.prepend(chemin);

    // Limiter la taille de la liste
    while (m_fichiersRecents.size() > MAX_FICHIERS_RECENTS) {
        m_fichiersRecents.removeLast();
    }

    emit fichiersRecentsModifies();
}

void Preferences::effacerFichiersRecents()
{
    m_fichiersRecents.clear();
    emit fichiersRecentsModifies();
}

bool Preferences::restaurerSession() const
{
    return m_restaurerSession;
}

void Preferences::setRestaurerSession(bool restaurer)
{
    if (m_restaurerSession != restaurer) {
        m_restaurerSession = restaurer;
        emit restaurerSessionModifie(restaurer);
    }
}

void Preferences::charger()
{
    QSettings settings;

    // Langue
    m_langue = settings.value("general/langue", m_langue).toString();

    // Interface
    m_modeNuit = settings.value("interface/modeNuit", m_modeNuit).toBool();
    m_couleurPrimaire = settings.value("interface/couleurPrimaire", m_couleurPrimaire).value<QColor>();

    // Éditeur
    m_policeEditeur = settings.value("editeur/police", m_policeEditeur).value<QFont>();
    m_tailleTabulation = settings.value("editeur/tailleTabulation", m_tailleTabulation).toInt();
    m_surlignageSyntaxe = settings.value("editeur/surlignageSyntaxe", m_surlignageSyntaxe).toBool();

    // Fichiers récents
    m_fichiersRecents = settings.value("fichiers/recents", m_fichiersRecents).toStringList();

    // Session
    m_restaurerSession = settings.value("session/restaurer", m_restaurerSession).toBool();

    qDebug() << "Préférences chargées";
}

void Preferences::sauvegarder()
{
    QSettings settings;

    // Langue
    settings.setValue("general/langue", m_langue);

    // Interface
    settings.setValue("interface/modeNuit", m_modeNuit);
    settings.setValue("interface/couleurPrimaire", m_couleurPrimaire);

    // Éditeur
    settings.setValue("editeur/police", m_policeEditeur);
    settings.setValue("editeur/tailleTabulation", m_tailleTabulation);
    settings.setValue("editeur/surlignageSyntaxe", m_surlignageSyntaxe);

    // Fichiers récents
    settings.setValue("fichiers/recents", m_fichiersRecents);

    // Session
    settings.setValue("session/restaurer", m_restaurerSession);

    // Force l'écriture sur le disque
    settings.sync();

    qDebug() << "Préférences sauvegardées";
}

void Preferences::restaurerDefauts()
{
    // Réinitialiser tous les paramètres à leurs valeurs par défaut
    m_langue = "fr";
    m_modeNuit = false;
    m_couleurPrimaire = QColor(0, 120, 215);
    m_policeEditeur = QFont("Consolas", 11);
    m_tailleTabulation = 4;
    m_surlignageSyntaxe = true;
    m_fichiersRecents.clear();
    m_restaurerSession = true;

    // Émettre tous les signaux
    emit langueModifiee(m_langue);
    emit modeNuitModifie(m_modeNuit);
    emit couleurPrimaireModifiee(m_couleurPrimaire);
    emit policeEditeurModifiee(m_policeEditeur);
    emit tailleTabulationModifiee(m_tailleTabulation);
    emit surlignageSyntaxeModifie(m_surlignageSyntaxe);
    emit fichiersRecentsModifies();
    emit restaurerSessionModifie(m_restaurerSession);

    qDebug() << "Préférences réinitialisées aux valeurs par défaut";
}
```

## Exemple d'utilisation : Sauvegarde de la géométrie des fenêtres

QSettings est particulièrement utile pour sauvegarder l'état et la position des fenêtres :

```cpp
// Dans le constructeur de la fenêtre
MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent)
{
    // Configuration de l'interface...

    // Restauration de la géométrie et de l'état
    QSettings settings;
    restoreGeometry(settings.value("mainWindow/geometry").toByteArray());
    restoreState(settings.value("mainWindow/windowState").toByteArray());

    // Autres initialisations...
}

// Dans l'événement de fermeture
void MainWindow::closeEvent(QCloseEvent *event)
{
    // Sauvegarde de la géométrie et de l'état
    QSettings settings;
    settings.setValue("mainWindow/geometry", saveGeometry());
    settings.setValue("mainWindow/windowState", saveState());

    QMainWindow::closeEvent(event);
}
```

## Visualisation et édition manuelle des paramètres

Selon la plateforme, vous pouvez accéder aux paramètres stockés pour les visualiser ou les modifier :

- **Windows** : Éditeur de registre (regedit.exe) -> `HKEY_CURRENT_USER\Software\[Société]\[Application]`
- **macOS** : Fichiers .plist dans `~/Library/Preferences/[com.societe.application].plist`
- **Linux** : Fichiers INI dans `~/.config/[societe]/[application].conf`

Pour les fichiers INI (utilisables sur toutes les plateformes), le format est simple et lisible :

```ini
[General]
langue=fr

[interface]
modeNuit=true
couleurPrimaire=@Variant(\0\0\0\x43\0\0\0\0\0\0\0\xd7\0\0\0\0)

[editeur]
tailleTabulation=4
surlignageSyntaxe=true
```

## Bonnes pratiques pour QSettings

1. **Définir les valeurs par défaut** lors de la lecture pour gérer le premier lancement
2. **Organiser les paramètres** en groupes logiques
3. **Centraliser la gestion** dans une classe dédiée (comme l'exemple Preferences)
4. **Utiliser sync()** pour les modifications critiques
5. **Vérifier le statut** après les opérations importantes
6. **Éviter de stocker des données volumineuses** (préférer les autres méthodes pour cela)
7. **Considérer la portée** (UserScope vs SystemScope) selon les besoins

## Quand utiliser QSettings vs autres méthodes

| Type de données | Méthode recommandée |
|-----------------|---------------------|
| Préférences utilisateur | QSettings |
| État de l'interface | QSettings |
| Petite quantité de données structurées | QSettings ou JSON |
| Grandes quantités de données | Base de données (SQL) |
| Objets Qt complexes | QDataStream |
| Données à partager avec d'autres systèmes | JSON ou XML |
| Documents hiérarchiques | XML |

## Conclusion

QSettings offre une solution simple et efficace pour stocker les paramètres d'application et autres données légères. Sa simplicité d'utilisation, son adaptation automatique aux différentes plateformes et sa performance en font le choix idéal pour gérer les préférences utilisateur et l'état de l'interface.

Pour l'utilisateur final, cela permet une expérience fluide où ses préférences sont automatiquement sauvegardées et restaurées à chaque utilisation de l'application.

Dans ce chapitre sur la persistance des données, nous avons exploré différentes méthodes, de la plus simple (QSettings) à la plus complexe (bases de données SQL), en passant par des formats intermédiaires (QDataStream, JSON, XML). Chaque méthode a ses forces et ses faiblesses, et le choix dépend des besoins spécifiques de votre application.

⏭️ [Communication réseau](/05-communication-reseau)
