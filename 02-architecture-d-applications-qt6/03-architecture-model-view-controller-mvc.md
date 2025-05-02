# 2.3 Architecture Model-View-Controller (MVC)

🔝 Retour à la [Table des matières](/SOMMAIRE.md)

L'architecture Model-View-Controller (MVC) est un modèle de conception fondamental dans le développement d'applications modernes. Qt6 implémente une variante de ce modèle, parfois appelée architecture Model-View, qui vous aidera à créer des applications bien structurées et faciles à maintenir.

## Qu'est-ce que l'architecture MVC ?

L'architecture MVC divise votre application en trois composants principaux :

![Architecture MVC](illustration_mvc.png)

1. **Le Modèle (Model)** : Contient les données et la logique métier
2. **La Vue (View)** : Affiche les données à l'utilisateur
3. **Le Contrôleur (Controller)** : Gère les interactions et fait le lien entre le modèle et la vue

Cette séparation présente plusieurs avantages :
- **Clarté du code** : Chaque partie a un rôle bien défini
- **Réutilisabilité** : Le même modèle peut être utilisé avec différentes vues
- **Facilité de maintenance** : Les modifications dans une partie n'affectent pas les autres
- **Développement parallèle** : Différentes équipes peuvent travailler sur chaque composant

## Architecture MVC dans Qt

Qt implémente une variante de l'architecture MVC, souvent appelée architecture **Modèle-Vue** car le rôle du contrôleur est partiellement intégré dans la vue.

![Architecture Modèle-Vue dans Qt](illustration_modele_vue_qt.png)

### Les modèles dans Qt

Les modèles dans Qt dérivent généralement de la classe `QAbstractItemModel` ou de ses sous-classes :

- **QStringListModel** : Pour les listes simples de chaînes de caractères
- **QStandardItemModel** : Pour les structures de données hiérarchiques génériques
- **QSqlTableModel** : Pour les tables de bases de données SQL
- **QFileSystemModel** : Pour le système de fichiers

Voici un exemple simple d'utilisation de `QStringListModel` :

```cpp
#include <QApplication>
#include <QListView>
#include <QStringListModel>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création des données
    QStringList listeNoms;
    listeNoms << "Alice" << "Bob" << "Charlie" << "David" << "Eva";

    // Création du modèle
    QStringListModel modele;
    modele.setStringList(listeNoms);

    // Création de la vue
    QListView vue;
    vue.setModel(&modele);  // Connexion de la vue au modèle
    vue.show();

    return app.exec();
}
```

### Les vues dans Qt

Les vues sont des widgets qui affichent les données d'un modèle. Qt propose plusieurs vues standard :

- **QListView** : Affiche les données sous forme de liste
- **QTableView** : Affiche les données sous forme de tableau
- **QTreeView** : Affiche les données sous forme d'arborescence
- **QColumnView** : Affiche les données dans plusieurs colonnes liées

Ces vues partagent une interface commune et peuvent être utilisées avec n'importe quel modèle compatible.

### Les délégués dans Qt

Dans l'architecture Modèle-Vue de Qt, les délégués jouent un rôle similaire à celui du contrôleur. Un délégué :

- Détermine comment les éléments sont affichés
- Gère l'édition des données
- Traite les interactions de l'utilisateur avec les éléments individuels

Qt fournit une classe de base `QStyledItemDelegate` que vous pouvez étendre pour personnaliser l'apparence et le comportement des éléments.

## Création d'un modèle personnalisé

Pour créer un modèle personnalisé, vous devez généralement hériter de `QAbstractItemModel` ou d'une de ses sous-classes comme `QAbstractListModel`.

Voici un exemple simple de modèle personnalisé pour une liste de tâches :

```cpp
// tachemodel.h
class TacheModel : public QAbstractListModel
{
    Q_OBJECT
public:
    // Structure pour représenter une tâche
    struct Tache {
        QString description;
        bool terminee;
    };

    // Constructeur
    explicit TacheModel(QObject *parent = nullptr);

    // Méthodes obligatoires de QAbstractListModel
    int rowCount(const QModelIndex &parent = QModelIndex()) const override;
    QVariant data(const QModelIndex &index, int role = Qt::DisplayRole) const override;
    bool setData(const QModelIndex &index, const QVariant &value, int role = Qt::EditRole) override;
    Qt::ItemFlags flags(const QModelIndex &index) const override;

    // Méthodes spécifiques à notre modèle
    void ajouterTache(const QString &description);
    void supprimerTache(int index);

private:
    QList<Tache> m_taches;  // Liste des tâches
};

// tachemodel.cpp
int TacheModel::rowCount(const QModelIndex &parent) const
{
    // Retourne le nombre de tâches dans le modèle
    return m_taches.count();
}

QVariant TacheModel::data(const QModelIndex &index, int role) const
{
    // Vérifie si l'index est valide
    if (!index.isValid() || index.row() >= m_taches.size())
        return QVariant();

    // Récupère la tâche correspondant à l'index
    const Tache &tache = m_taches[index.row()];

    // Retourne les données en fonction du rôle demandé
    switch (role) {
    case Qt::DisplayRole:
        return tache.description;
    case Qt::CheckStateRole:
        return tache.terminee ? Qt::Checked : Qt::Unchecked;
    default:
        return QVariant();
    }
}

bool TacheModel::setData(const QModelIndex &index, const QVariant &value, int role)
{
    // Vérifie si l'index est valide et si le rôle est approprié
    if (!index.isValid() || index.row() >= m_taches.size())
        return false;

    // Modifie les données en fonction du rôle
    if (role == Qt::CheckStateRole) {
        // Change l'état de la tâche (terminée ou non)
        m_taches[index.row()].terminee = value.toBool();

        // Émet le signal dataChanged pour indiquer que les données ont changé
        emit dataChanged(index, index, {role});
        return true;
    }

    return false;
}

Qt::ItemFlags TacheModel::flags(const QModelIndex &index) const
{
    // Les éléments sont activés, sélectionnables et cochables
    return Qt::ItemIsEnabled | Qt::ItemIsSelectable | Qt::ItemIsUserCheckable;
}

void TacheModel::ajouterTache(const QString &description)
{
    // Démarre l'insertion d'une nouvelle ligne
    beginInsertRows(QModelIndex(), m_taches.size(), m_taches.size());

    // Ajoute la nouvelle tâche
    m_taches.append({description, false});

    // Termine l'insertion
    endInsertRows();
}

void TacheModel::supprimerTache(int index)
{
    // Vérifie si l'index est valide
    if (index < 0 || index >= m_taches.size())
        return;

    // Démarre la suppression d'une ligne
    beginRemoveRows(QModelIndex(), index, index);

    // Supprime la tâche
    m_taches.removeAt(index);

    // Termine la suppression
    endRemoveRows();
}
```

## Utilisation de modèles et vues dans une application

Voici un exemple complet d'application de liste de tâches utilisant le modèle que nous venons de créer :

```cpp
#include <QApplication>
#include <QMainWindow>
#include <QListView>
#include <QPushButton>
#include <QLineEdit>
#include <QHBoxLayout>
#include <QVBoxLayout>
#include "tachemodel.h"

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);

    // Création de la fenêtre principale
    QMainWindow mainWindow;
    mainWindow.setWindowTitle("Liste de tâches");

    // Création du widget central
    QWidget *centralWidget = new QWidget(&mainWindow);
    mainWindow.setCentralWidget(centralWidget);

    // Création du modèle
    TacheModel *modele = new TacheModel(centralWidget);

    // Ajout de quelques tâches de démonstration
    modele->ajouterTache("Apprendre Qt6");
    modele->ajouterTache("Comprendre l'architecture MVC");

    // Création de la vue
    QListView *vue = new QListView(centralWidget);
    vue->setModel(modele);

    // Création des widgets pour ajouter des tâches
    QLineEdit *champTache = new QLineEdit(centralWidget);
    QPushButton *boutonAjouter = new QPushButton("Ajouter", centralWidget);

    // Connexion du bouton à une lambda qui ajoute une tâche
    QObject::connect(boutonAjouter, &QPushButton::clicked, [modele, champTache]() {
        if (!champTache->text().isEmpty()) {
            modele->ajouterTache(champTache->text());
            champTache->clear();
        }
    });

    // Création du bouton de suppression
    QPushButton *boutonSupprimer = new QPushButton("Supprimer", centralWidget);

    // Connexion du bouton à une lambda qui supprime la tâche sélectionnée
    QObject::connect(boutonSupprimer, &QPushButton::clicked, [modele, vue]() {
        QModelIndex index = vue->currentIndex();
        if (index.isValid()) {
            modele->supprimerTache(index.row());
        }
    });

    // Création des layouts
    QHBoxLayout *layoutBoutons = new QHBoxLayout();
    layoutBoutons->addWidget(champTache);
    layoutBoutons->addWidget(boutonAjouter);
    layoutBoutons->addWidget(boutonSupprimer);

    QVBoxLayout *layoutPrincipal = new QVBoxLayout(centralWidget);
    layoutPrincipal->addWidget(vue);
    layoutPrincipal->addLayout(layoutBoutons);

    // Affichage de la fenêtre
    mainWindow.resize(400, 300);
    mainWindow.show();

    return app.exec();
}
```

## Les modèles prédéfinis dans Qt

Pour des cas d'utilisation courants, Qt propose des modèles prêts à l'emploi :

### QStringListModel

Pour représenter une liste simple de chaînes de caractères :

```cpp
QStringList listeNoms = {"Alice", "Bob", "Charlie"};
QStringListModel *modele = new QStringListModel(listeNoms);
QListView *vue = new QListView();
vue->setModel(modele);
```

### QStandardItemModel

Pour des structures de données plus complexes ou hiérarchiques :

```cpp
QStandardItemModel *modele = new QStandardItemModel(4, 2);
modele->setHeaderData(0, Qt::Horizontal, "Nom");
modele->setHeaderData(1, Qt::Horizontal, "Âge");

// Remplissage du modèle
modele->setData(modele->index(0, 0), "Alice");
modele->setData(modele->index(0, 1), 25);
modele->setData(modele->index(1, 0), "Bob");
modele->setData(modele->index(1, 1), 30);

// Utilisation avec une vue
QTableView *vue = new QTableView();
vue->setModel(modele);
```

### QSqlTableModel

Pour travailler avec des bases de données SQL :

```cpp
QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
db.setDatabaseName("mabase.sqlite");
db.open();

QSqlTableModel *modele = new QSqlTableModel();
modele->setTable("utilisateurs");
modele->select();

QTableView *vue = new QTableView();
vue->setModel(modele);
```

## Avantages de l'architecture MVC en pratique

### Exemple concret : changement de vue

Un des grands avantages de l'architecture MVC est la possibilité de changer facilement de vue sans toucher au modèle :

```cpp
// Notre modèle de tâches
TacheModel *modele = new TacheModel();
modele->ajouterTache("Tâche 1");
modele->ajouterTache("Tâche 2");

// Affichage dans une liste
QListView *listView = new QListView();
listView->setModel(modele);
listView->show();

// Affichage dans un tableau (avec le même modèle)
QTableView *tableView = new QTableView();
tableView->setModel(modele);
tableView->show();
```

### Modification des données centralisée

Avec l'architecture MVC, les modifications des données se font à un seul endroit (le modèle) et sont automatiquement reflétées dans toutes les vues :

```cpp
// Si nous modifions le modèle...
modele->ajouterTache("Nouvelle tâche");

// Toutes les vues connectées à ce modèle seront automatiquement mises à jour
```

## Utilisation de MVC avec QML

L'architecture MVC peut également être utilisée avec QML. Voici un exemple simple :

```cpp
// Modèle C++
class TacheModel : public QAbstractListModel
{
    Q_OBJECT
public:
    // ... (comme dans l'exemple précédent)

    // Exposition des rôles pour QML
    QHash<int, QByteArray> roleNames() const override
    {
        QHash<int, QByteArray> roles;
        roles[Qt::DisplayRole] = "description";
        roles[Qt::CheckStateRole] = "terminee";
        return roles;
    }
};
```

```qml
// Vue QML
import QtQuick 2.15
import QtQuick.Controls 2.15

ListView {
    width: 300
    height: 400
    model: tacheModel  // Le modèle exposé depuis C++

    delegate: Rectangle {
        width: parent.width
        height: 50

        Row {
            spacing: 10

            CheckBox {
                checked: model.terminee === Qt::Checked
                onClicked: tacheModel.setData(tacheModel.index(index, 0), checked, Qt.CheckStateRole)
            }

            Text {
                text: model.description
                font.pixelSize: 16
                anchors.verticalCenter: parent.verticalCenter
            }
        }
    }
}
```

## Conseils pour les débutants

### Par où commencer ?

1. **Commencez simplement** : Utilisez d'abord les modèles prédéfinis comme `QStringListModel`
2. **Comprenez les rôles** : Les données dans un modèle peuvent avoir différents rôles (affichage, édition, etc.)
3. **Explorez les vues standard** : Familiarisez-vous avec `QListView`, `QTableView` et `QTreeView`
4. **Créez ensuite vos propres modèles** : Lorsque vous êtes à l'aise, créez des modèles personnalisés

### Erreurs courantes à éviter

- **Ne pas émettre les signaux appropriés** : Oublier d'appeler `beginInsertRows`/`endInsertRows` ou `dataChanged`
- **Confondre index de modèle et index de ligne** : Un `QModelIndex` n'est pas un simple entier
- **Ignorer les performances** : Les méthodes comme `data()` sont appelées très fréquemment, elles doivent être optimisées

## Conclusion

L'architecture MVC (ou Modèle-Vue dans Qt) est un concept puissant qui vous aide à structurer votre application de manière propre et maintenable. En séparant les données (modèle) de leur présentation (vue), vous obtenez un code plus flexible et plus facile à faire évoluer.

Qt6 fournit un cadre riche pour implémenter cette architecture, avec des classes de base pour les modèles et une variété de vues prêtes à l'emploi. Maîtriser ces concepts vous permettra de créer des applications robustes et évolutives, capables de gérer efficacement des données complexes.

Dans la prochaine section, nous explorerons l'organisation modulaire du code dans Qt6, une autre approche essentielle pour maintenir la qualité de vos applications au fur et à mesure qu'elles grandissent.

⏭️ [Organisation modulaire du code](/02-architecture-d-applications-qt6/04-organisation-modulaire-du-code.md)
