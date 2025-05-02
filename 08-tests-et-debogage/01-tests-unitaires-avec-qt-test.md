# 8.1 Tests unitaires avec Qt Test

Les tests unitaires sont une pratique fondamentale en développement logiciel qui consiste à tester individuellement des parties isolées de votre code (généralement des fonctions ou des méthodes) pour vérifier qu'elles fonctionnent correctement. Qt fournit un framework de test unitaire appelé **Qt Test** qui est spécialement conçu pour tester des applications Qt.

## Pourquoi faire des tests unitaires ?

Avant de nous plonger dans le fonctionnement de Qt Test, comprenons pourquoi les tests unitaires sont importants :

- Ils détectent les bugs tôt dans le cycle de développement
- Ils vous permettent de modifier votre code avec confiance
- Ils servent de documentation sur le comportement attendu de votre code
- Ils encouragent un meilleur design de code (un code facile à tester est souvent un code bien conçu)

## Installation et configuration

La bonne nouvelle, c'est que Qt Test est inclus dans l'installation standard de Qt. Pour l'utiliser, vous avez seulement besoin d'ajouter la ligne suivante à votre fichier `.pro` :

```
QT += testlib
```

Si vous utilisez CMake, ajoutez ceci à votre fichier `CMakeLists.txt` :

```cmake
find_package(Qt6 COMPONENTS Test REQUIRED)
target_link_libraries(your_target_name PRIVATE Qt6::Test)
```

## Création d'un projet de test simple

Commençons par créer un projet de test simple avec Qt Creator :

1. Lancez Qt Creator
2. Créez un nouveau projet (Fichier → Nouveau fichier ou projet)
3. Sélectionnez "Applications" → "Projet d'application Qt Test"
4. Suivez l'assistant pour configurer votre projet

Votre projet contiendra une classe de test générée automatiquement qui ressemble à ceci :

```cpp
#include <QtTest>

class TestExample : public QObject
{
    Q_OBJECT

public:
    TestExample();
    ~TestExample();

private slots:
    void initTestCase();
    void cleanupTestCase();
    void test_case1();
};

TestExample::TestExample()
{
}

TestExample::~TestExample()
{
}

void TestExample::initTestCase()
{
    // Appelé avant le premier test
}

void TestExample::cleanupTestCase()
{
    // Appelé après le dernier test
}

void TestExample::test_case1()
{
    QVERIFY(true); // Un test qui réussit toujours
}

QTEST_APPLESS_MAIN(TestExample)
#include "test_example.moc"
```

## Structure d'une classe de test

Analysons la structure typique d'une classe de test Qt :

1. **Déclaration de classe** : Votre classe de test doit hériter de `QObject` et utiliser la macro `Q_OBJECT`.

2. **Méthodes de test** : Chaque méthode de test doit être déclarée comme un slot privé. Ces méthodes seront exécutées automatiquement par le framework.

3. **Méthodes spéciales** :
   - `initTestCase()` : Exécutée une seule fois avant tous les tests
   - `cleanupTestCase()` : Exécutée une seule fois après tous les tests
   - `init()` : Exécutée avant chaque test
   - `cleanup()` : Exécutée après chaque test

4. **Macro principale** : `QTEST_APPLESS_MAIN` ou `QTEST_MAIN` pour générer la fonction `main()`.

## Les macros de vérification essentielles

Qt Test fournit plusieurs macros pour vérifier différentes conditions :

- **QVERIFY(condition)** : Vérifie qu'une condition est vraie
```cpp
QVERIFY(2 + 2 == 4);
```

- **QCOMPARE(actual, expected)** : Compare deux valeurs
```cpp
QCOMPARE(calculerSomme(2, 2), 4);
```

- **QCOMPARE_EQ(actual, expected)** : Vérifie l'égalité (utilise `==` pour la comparaison)
```cpp
QString str = "Hello";
QCOMPARE_EQ(str, QString("Hello"));
```

- **QVERIFY_EXCEPTION_THROWN(expression, exceptionType)** : Vérifie qu'une expression lance une exception
```cpp
QVERIFY_EXCEPTION_THROWN(diviserParZero(), std::invalid_argument);
```

- **QFAIL(message)** : Fait échouer le test avec un message
```cpp
if (valeurInattendue)
    QFAIL("Une erreur inattendue s'est produite");
```

## Exemple pratique : Tester une classe simple

Imaginons que nous avons une classe `Calculatrice` simple :

```cpp
// calculatrice.h
class Calculatrice
{
public:
    int additionner(int a, int b);
    int soustraire(int a, int b);
    int multiplier(int a, int b);
    double diviser(int a, int b);
};

// calculatrice.cpp
int Calculatrice::additionner(int a, int b)
{
    return a + b;
}

int Calculatrice::soustraire(int a, int b)
{
    return a - b;
}

int Calculatrice::multiplier(int a, int b)
{
    return a * b;
}

double Calculatrice::diviser(int a, int b)
{
    if (b == 0)
        throw std::invalid_argument("Division par zéro");
    return static_cast<double>(a) / b;
}
```

Voici comment nous pourrions écrire des tests pour cette classe :

```cpp
#include <QtTest>
#include "calculatrice.h"

class TestCalculatrice : public QObject
{
    Q_OBJECT

private:
    Calculatrice calc;

private slots:
    void test_additionner_data();
    void test_additionner();
    void test_soustraire();
    void test_multiplier();
    void test_diviser();
    void test_diviser_par_zero();
};

void TestCalculatrice::test_additionner_data()
{
    // Définition des données de test
    QTest::addColumn<int>("a");
    QTest::addColumn<int>("b");
    QTest::addColumn<int>("resultat");

    // Ajout des données de test
    QTest::newRow("positifs") << 2 << 3 << 5;
    QTest::newRow("négatif+positif") << -2 << 3 << 1;
    QTest::newRow("négatifs") << -2 << -3 << -5;
    QTest::newRow("zéros") << 0 << 0 << 0;
}

void TestCalculatrice::test_additionner()
{
    // Récupération des données de test
    QFETCH(int, a);
    QFETCH(int, b);
    QFETCH(int, resultat);

    // Test avec les données
    QCOMPARE(calc.additionner(a, b), resultat);
}

void TestCalculatrice::test_soustraire()
{
    QCOMPARE(calc.soustraire(5, 3), 2);
    QCOMPARE(calc.soustraire(3, 5), -2);
    QCOMPARE(calc.soustraire(-3, -5), 2);
}

void TestCalculatrice::test_multiplier()
{
    QCOMPARE(calc.multiplier(2, 3), 6);
    QCOMPARE(calc.multiplier(2, -3), -6);
    QCOMPARE(calc.multiplier(0, 5), 0);
}

void TestCalculatrice::test_diviser()
{
    QCOMPARE(calc.diviser(6, 3), 2.0);
    QCOMPARE(calc.diviser(5, 2), 2.5);
    QCOMPARE(calc.diviser(0, 5), 0.0);
}

void TestCalculatrice::test_diviser_par_zero()
{
    QVERIFY_EXCEPTION_THROWN(calc.diviser(5, 0), std::invalid_argument);
}

QTEST_APPLESS_MAIN(TestCalculatrice)
#include "testcalculatrice.moc"
```

## Tests avec des données (Data-Driven Tests)

Dans l'exemple précédent, vous avez vu comment utiliser des tests pilotés par les données avec les fonctions `test_additionner_data()` et `test_additionner()`.

Cette approche est très puissante car elle vous permet de :
1. Définir de nombreux cas de test sans dupliquer le code
2. Donner des noms explicites à chaque cas de test
3. Ajouter facilement de nouveaux cas de test

Le modèle à suivre est :
1. Créer une méthode `test_something_data()` qui définit les colonnes et les données
2. Créer une méthode `test_something()` qui récupère les données avec `QFETCH` et effectue les vérifications

## Tester les interfaces graphiques

Qt Test inclut également des fonctionnalités pour tester les interfaces graphiques :

```cpp
#include <QtTest>
#include <QLineEdit>
#include <QPushButton>

class TestIHM : public QObject
{
    Q_OBJECT

private slots:
    void testLineEdit();
    void testButton();
};

void TestIHM::testLineEdit()
{
    QLineEdit lineEdit;

    // Simuler la saisie de texte
    QTest::keyClicks(&lineEdit, "Hello");

    QCOMPARE(lineEdit.text(), QString("Hello"));
}

void TestIHM::testButton()
{
    QPushButton button("Cliquez-moi");

    // Variable pour vérifier si le signal clicked a été émis
    bool clicked = false;
    connect(&button, &QPushButton::clicked, [&clicked]() {
        clicked = true;
    });

    // Simuler un clic
    QTest::mouseClick(&button, Qt::LeftButton);

    // Vérifier que le signal a été émis
    QVERIFY(clicked);
}

QTEST_MAIN(TestIHM)
#include "testihm.moc"
```

## Tester les signaux et slots

Qt Test permet également de tester l'émission des signaux avec la classe `QSignalSpy` :

```cpp
#include <QtTest>
#include <QSignalSpy>
#include <QPushButton>

class TestSignaux : public QObject
{
    Q_OBJECT

private slots:
    void testSignalClick();
};

void TestSignaux::testSignalClick()
{
    QPushButton button;

    // Créer un espion pour le signal clicked
    QSignalSpy spy(&button, &QPushButton::clicked);

    // Simuler un clic
    QTest::mouseClick(&button, Qt::LeftButton);

    // Vérifier que le signal a été émis une fois
    QCOMPARE(spy.count(), 1);

    // Simuler un autre clic
    QTest::mouseClick(&button, Qt::LeftButton);

    // Vérifier que le signal a été émis une deuxième fois
    QCOMPARE(spy.count(), 2);
}

QTEST_MAIN(TestSignaux)
#include "testsignaux.moc"
```

## Exécution des tests

Pour exécuter les tests :

1. **Depuis Qt Creator** : Cliquez sur le bouton de lecture (Run) ou utilisez Ctrl+R
2. **Depuis la ligne de commande** : Exécutez le fichier exécutable généré

Les résultats apparaîtront dans la sortie de la console :

```
********* Start testing of TestCalculatrice *********
Config: Using QtTest library 6.5.1, Qt 6.5.1
PASS   : TestCalculatrice::initTestCase()
PASS   : TestCalculatrice::test_additionner(positifs)
PASS   : TestCalculatrice::test_additionner(négatif+positif)
PASS   : TestCalculatrice::test_additionner(négatifs)
PASS   : TestCalculatrice::test_additionner(zéros)
PASS   : TestCalculatrice::test_soustraire()
PASS   : TestCalculatrice::test_multiplier()
PASS   : TestCalculatrice::test_diviser()
PASS   : TestCalculatrice::test_diviser_par_zero()
PASS   : TestCalculatrice::cleanupTestCase()
Totals: 10 passed, 0 failed, 0 skipped, 0 blacklisted, 0ms
********* Finished testing of TestCalculatrice *********
```

## Options d'exécution des tests

Qt Test offre plusieurs options pour personnaliser l'exécution des tests :

```
./test_calculatrice -help          # Afficher l'aide
./test_calculatrice -v2            # Mode verbeux (plus de détails)
./test_calculatrice -silent        # Mode silencieux
./test_calculatrice -functions     # Liste toutes les fonctions de test
./test_calculatrice testFunction   # Exécute uniquement la fonction spécifiée
```

## Bonnes pratiques pour les tests unitaires

1. **Tests indépendants** : Chaque test devrait être indépendant des autres tests.

2. **Tests rapides** : Les tests unitaires doivent s'exécuter rapidement pour être utilisés fréquemment.

3. **Nommage explicite** : Donnez des noms clairs à vos tests qui décrivent ce qu'ils testent.

4. **Une assertion par test** : Idéalement, chaque test ne devrait vérifier qu'une seule chose.

5. **Couverture de code** : Visez à tester toutes les branches importantes de votre code.

6. **Tests des cas limites** : Testez les valeurs limites et les cas spéciaux.

7. **Intégration continue** : Exécutez vos tests automatiquement à chaque commit.

## Conclusion

Qt Test fournit un framework puissant et facile à utiliser pour les tests unitaires d'applications Qt. En intégrant les tests unitaires dans votre processus de développement, vous améliorerez considérablement la qualité de votre code et réduirez le temps passé à déboguer des problèmes.

Dans la section suivante, nous explorerons les outils de débogage intégrés à Qt Creator, qui vous aideront à identifier et résoudre les problèmes dans votre code.
