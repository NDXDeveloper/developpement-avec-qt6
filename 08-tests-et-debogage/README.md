# 8. Tests et débogage

Le test et le débogage sont des étapes essentielles dans le développement d'applications robustes. Qt6 fournit un ensemble d'outils puissants qui vous aideront à identifier, comprendre et résoudre les problèmes dans votre code. Cette section couvre les concepts fondamentaux du test et du débogage spécifiques à Qt6.

## Pourquoi tester et déboguer ?

Avant de plonger dans les outils, il est important de comprendre pourquoi le test et le débogage sont cruciaux :

- **Qualité du code** : Les tests garantissent que votre application fonctionne comme prévu.
- **Détection précoce des bugs** : Plus un bug est détecté tôt, moins il est coûteux à corriger.
- **Refactoring en toute confiance** : Les tests vous permettent de modifier votre code sans craindre de casser des fonctionnalités existantes.
- **Documentation vivante** : Les tests agissent comme une documentation qui décrit le comportement attendu de votre code.

## L'approche Qt pour le débogage

Qt Creator, l'IDE principal pour le développement Qt, intègre un débogueur puissant qui fonctionne avec différents compilateurs (GCC, MSVC, Clang). Voici les fondamentaux que tout développeur Qt devrait connaître :

### Points d'arrêt (Breakpoints)

Les points d'arrêt sont l'outil le plus fondamental pour le débogage. Ils permettent de suspendre l'exécution de votre programme à un point spécifique.

**Comment définir des points d'arrêt dans Qt Creator :**
1. Cliquez simplement dans la marge gauche à côté du numéro de ligne où vous souhaitez arrêter l'exécution.
2. Un point rouge apparaît, indiquant votre point d'arrêt.
3. Vous pouvez également définir des points d'arrêt conditionnels en faisant un clic droit sur le point d'arrêt et en sélectionnant "Edit breakpoint".

![Exemple de point d'arrêt dans Qt Creator](https://doc.qt.io/qtcreator/images/qtcreator-breakpoints.png)

### Inspection des variables

Lorsque votre programme est en pause à un point d'arrêt, vous pouvez examiner les valeurs des variables :

- Le panneau **Locals & Expressions** affiche automatiquement toutes les variables locales.
- Les variables membres de la classe actuelle apparaissent dans une section séparée.
- Vous pouvez ajouter des expressions à surveiller en cliquant sur le bouton "+" dans le panneau **Watchers**.

### Navigation dans le code pendant le débogage

Qt Creator offre plusieurs façons de contrôler l'exécution de votre programme lors du débogage :

- **Step Over (F10)** : Exécute la ligne actuelle sans entrer dans les fonctions.
- **Step Into (F11)** : Entre dans la fonction appelée sur la ligne actuelle.
- **Step Out (Shift+F11)** : Continue l'exécution jusqu'à la fin de la fonction actuelle.
- **Continue (F5)** : Reprend l'exécution jusqu'au prochain point d'arrêt.

### Console de débogage

La console de débogage affiche les sorties standard et d'erreur de votre application. C'est particulièrement utile pour voir les messages de `qDebug()`, `qWarning()`, `qCritical()` et `qFatal()`.

## Utilisation de qDebug pour le débogage

Qt fournit un système pratique pour générer des messages de débogage à travers la macro `qDebug()` et ses compagnons :

```cpp
#include <QDebug>

void maFonction() {
    qDebug() << "Démarrage de la fonction";

    int x = 5;
    qDebug() << "La valeur de x est" << x;

    if(x > 10) {
        qWarning() << "Attention : x est supérieur à 10";
    }

    // Une erreur critique
    if(x < 0) {
        qCritical() << "Erreur : x ne peut pas être négatif";
        return;
    }

    qDebug() << "Fin de la fonction";
}
```

### Personnalisation des messages de débogage

Vous pouvez personnaliser la façon dont les messages de débogage sont formatés et traités en définissant un gestionnaire de messages :

```cpp
void myMessageHandler(QtMsgType type, const QMessageLogContext &context, const QString &msg) {
    // Formatez et redirigez les messages comme vous le souhaitez
    QByteArray localMsg = msg.toLocal8Bit();
    const char *file = context.file ? context.file : "";
    const char *function = context.function ? context.function : "";

    switch (type) {
    case QtDebugMsg:
        fprintf(stderr, "Debug: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        break;
    case QtInfoMsg:
        fprintf(stderr, "Info: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        break;
    case QtWarningMsg:
        fprintf(stderr, "Warning: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        break;
    case QtCriticalMsg:
        fprintf(stderr, "Critical: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        break;
    case QtFatalMsg:
        fprintf(stderr, "Fatal: %s (%s:%u, %s)\n", localMsg.constData(), file, context.line, function);
        abort();
    }
}

int main(int argc, char *argv[]) {
    qInstallMessageHandler(myMessageHandler);

    QApplication app(argc, argv);
    // ...
    return app.exec();
}
```

### Mode de débogage vs Mode de production

Il est souvent utile de n'activer certains messages de débogage qu'en mode de développement. Qt fournit une macro `QT_DEBUG` qui n'est définie qu'en mode de débogage :

```cpp
#ifdef QT_DEBUG
    qDebug() << "Ce message n'apparaît qu'en mode debug";
#endif
```

## Débogage des interfaces graphiques

Le débogage des interfaces utilisateur pose des défis particuliers. Qt Creator inclut des outils spécifiques pour vous aider :

### Qt Creator Debug Helpers

Qt Creator comprend des helpers de débogage spéciaux pour les types Qt, qui affichent les objets Qt de manière conviviale :

- Les chaînes de caractères (QString) sont affichées comme du texte lisible.
- Les conteneurs (QList, QVector, QMap) sont affichés avec leur contenu.
- Les objets complexes comme QPixmap montrent un aperçu visuel.

### Débogage des signaux et slots

Les connexions signaux-slots sont au cœur de Qt, mais peuvent être difficiles à déboguer. Qt fournit des outils pour vous aider :

1. **Mode Debug de Qt** : Qt peut émettre des avertissements concernant les connexions de signaux et slots échouées.

```cpp
// Activez ceci au début de votre main()
QLoggingCategory::setFilterRules("qt.core.qobject.debug=true");
```

2. **Utilitaire QSignalSpy** : Pour tester si un signal a été émis (utile pour les tests unitaires) :

```cpp
#include <QSignalSpy>

// Surveille le signal "clicked" d'un bouton
QPushButton button;
QSignalSpy spy(&button, &QPushButton::clicked);

// Simulez un clic
button.click();

// Vérifiez si le signal a été émis
QCOMPARE(spy.count(), 1);
```

### Inspecteur d'objets

Qt Creator inclut un inspecteur d'objets qui vous permet d'examiner la hiérarchie des objets QObject à l'exécution :

1. Pendant le débogage, accédez à **Debug > Qt > QML/C++ Debug Helpers Status**.
2. Utilisez **Debug > Qt > Analyze Qt Object Hierarchy** pour voir la structure de vos objets.

## Bonnes pratiques de débogage avec Qt

1. **Utilisez les assertions** : Qt fournit Q_ASSERT et Q_ASSERT_X pour vérifier vos hypothèses :

```cpp
Q_ASSERT(index >= 0); // Vérifie que l'index est positif
Q_ASSERT_X(index < list.size(), "processItem", "Index hors limites"); // Version avec message
```

2. **Gardez un journal** : Utilisez qDebug() stratégiquement pour suivre le flux d'exécution.

3. **Utilisez des enums pour les codes d'erreur** :

```cpp
enum ErrorCode {
    NoError = 0,
    FileNotFound,
    PermissionDenied,
    NetworkError
};

void processFile(const QString &filename, ErrorCode &error) {
    if (!QFile::exists(filename)) {
        error = FileNotFound;
        qDebug() << "Erreur" << error << ": Fichier non trouvé";
        return;
    }
    // ...
}
```

4. **Déboguer les fuites mémoire** :
   - Activez l'option de ligne de commande `-debug-and-release` pour avoir des versions debug et release.
   - Utilisez des outils comme Valgrind (Linux/macOS) ou Application Verifier (Windows).

5. **Journalisation de l'application** : Pour les applications déployées, envisagez d'implémenter un système de journalisation :

```cpp
// Exemple simple de journalisation dans un fichier
QFile logFile("application.log");
if (logFile.open(QIODevice::WriteOnly | QIODevice::Append)) {
    QTextStream stream(&logFile);
    stream << QDateTime::currentDateTime().toString() << " - " << message << "\n";
    logFile.close();
}
```

## Conclusion

Le débogage est un art qui s'améliore avec la pratique. Qt6 fournit un ensemble riche d'outils pour vous aider à identifier et résoudre les problèmes dans votre code. En maîtrisant ces techniques, vous deviendrez plus efficace dans le développement d'applications Qt robustes et fiables.

Dans la prochaine section, nous explorerons les tests unitaires avec Qt Test, qui vous permettront d'automatiser la vérification du comportement de votre code.
