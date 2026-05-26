LAB 22 – Développement Android avec JNI (Java Native Interface)

Cours : Programmation Mobile – Android avec Java

Présentation

Ce laboratoire a pour objectif de construire une application Android nommée JNIDemo capable de communiquer avec du code natif C++ via JNI (Java Native Interface). L’application appellera plusieurs fonctions natives, enverra des paramètres Java vers C++, récupérera des résultats calculés côté natif, et apprendra à gérer correctement le chargement de la bibliothèque partagée .so.

Ce laboratoire permet également de comprendre pourquoi JNI est encore utilisé dans les applications modernes : calcul intensif, réutilisation de bibliothèques C/C++, encapsulation d’algorithmes sensibles, traitement temps réel, ou ajout de couches de résistance au reverse engineering.

Objectifs pédagogiques

À la fin de ce laboratoire, vous serez capable de :

- créer un projet Android avec support C++ ;
- comprendre le rôle du NDK, de CMake et de JNI ;
- déclarer et appeler des méthodes natives depuis Java ;
- manipuler des types simples et complexes entre Java et C++ ;
- gérer des erreurs fréquentes comme UnsatisfiedLinkError ;
- lire les logs natifs dans Logcat.

Ce que l’application fera

L’application JNIDemo réalisera quatre démonstrations :

1. appel d’une fonction native simple helloFromJNI(),
2. calcul natif d’un factoriel avec contrôle d’erreur,
3. inversion d’une chaîne de caractères envoyée depuis Java,
4. traitement d’un tableau int[] envoyé au natif.

Prérequis

Avant de commencer, vérifiez les éléments suivants :

- Android Studio installé ;
- SDK Android configuré ;
- NDK, CMake et LLDB disponibles dans le SDK Manager ;
- connaissances de base sur Activity, layout XML et Java.

Architecture du laboratoire

Le flux général est le suivant :

Java / MainActivity
→ appelle une méthode native
→ Android charge libnative-lib.so
→ JNI transmet l’appel au code C++
→ le code C++ exécute le traitement
→ le résultat est converti et renvoyé vers Java
→ l’interface Android affiche le résultat

Étapes de réalisation

Étape 1 – Créer le projet Android avec support C++

Ouvrir Android Studio puis :
File → New Project → Empty Views Activity
Configurer :
Name : JNIDemo
Language : Java
Minimum SDK : API 24
Cocher Include C++ support
Build system : CMake

Étape 2 – Comprendre les rôles des composants

- JNI : interface entre Java/Kotlin et C/C++.
- NDK : ensemble d’outils pour utiliser C/C++ dans Android.
- CMake : outil de build pour compiler la bibliothèque native.
- Bibliothèque partagée .so : chargée via System.loadLibrary("native-lib").

Étape 3 – Vérifier la configuration Gradle

Fichier app/build.gradle (extrait) :

android {
    namespace "com.example.jnidemo"
    compileSdk 35

    defaultConfig {
        applicationId "com.example.jnidemo"
        minSdk 24
        targetSdk 35
        versionCode 1
        versionName "1.0"
    }

    externalNativeBuild {
        cmake {
            path file("src/main/cpp/CMakeLists.txt")
        }
    }
}

Étape 4 – Configurer le fichier CMakeLists.txt

Fichier app/src/main/cpp/CMakeLists.txt :

cmake_minimum_required(VERSION 3.22.1)
project("jnidemo")

add_library(
        native-lib
        SHARED
        native-lib.cpp)

find_library(
        log-lib
        log)

target_link_libraries(
        native-lib
        ${log-lib})

Étape 5 – Écrire le code natif C++

Fichier app/src/main/cpp/native-lib.cpp (contenu complet fourni dans le laboratoire). Les fonctions natives sont :

- helloFromJNI : retourne une chaîne de bienvenue.
- factorial : calcule la factorielle avec gestion d’erreur (négatif, overflow).
- reverseString : inverse une chaîne Java.
- sumArray : somme les éléments d’un tableau int[].

Étape 6 – Déclarer les méthodes natives côté Java

Dans MainActivity.java, déclarez les méthodes natives :

public native String helloFromJNI();
public native int factorial(int n);
public native String reverseString(String input);
public native int sumArray(int[] array);

Chargez la bibliothèque dans un bloc static :

static {
    System.loadLibrary("native-lib");
}

Étape 7 – Créer le layout XML complet

Fichier res/layout/activity_main.xml avec ScrollView et TextView pour chaque résultat.

Étape 8 – Compiler et exécuter

Lancez l’application sur un émulateur ou un appareil réel.

Étape 9 – Vérifier les logs natifs dans Logcat

Filtre avec le tag JNI_DEMO. Vous devez voir les traces LOGI et LOGE émises depuis le code C++.

Résultats des tests

À l’exécution, l’application affiche les résultats suivants (conformes aux tests guidés) :

Test 1 factorial(10) → 3628800
Test 2 factorial(-5) → -1 (erreur entrée négative)
Test 3 factorial(20) → -2 (dépassement d’entier)
Test 4 reverseString("") → "" (chaîne vide)
Bonus : reverseString("Test JNI") → "tseT INJ"
Test 5 sumArray(new int[]{}) → 0
Bonus : sumArray(new int[]{10,20}) → 30

Débogage des erreurs fréquentes

- UnsatisfiedLinkError : vérifiez le nom de la bibliothèque, le package, la signature JNI.
- Erreur de compilation C++ : vérifiez les inclusions (#include <algorithm>, #include <climits>).
- Plantage sur chaîne : n’oubliez pas ReleaseStringUTFChars.

Pourquoi JNI peut être utile

- Calcul intensif (image, cryptographie, jeu)
- Réutilisation de bibliothèques C/C++ existantes (OpenCV, audio, etc.)
- Protection partielle de logique sensible
- Accès à des API natives Android

Bonnes pratiques

1. Réduire les allers-retours Java ↔ natif.
2. Garder une API native propre et peu nombreuse.
3. Libérer systématiquement les ressources JNI (Get... / Release...).
4. Journaliser avec modération.
5. Utiliser R8 avec des règles -keep si nécessaire.
6. Gérer les ABI cibles.

Extension recommandée du laboratoire

- Extension A : multiplication matricielle native.
- Extension B : détection de caractères interdits dans une chaîne.
- Extension C : mini benchmark Java vs C++.
- Extension D : enregistrement dynamique des fonctions natives avec RegisterNatives.

Résumé pédagogique

Ce laboratoire illustre l’intégration complète entre Java et C++ sous Android via JNI. Il montre la création d’un projet avec support natif, l’écriture d’un code C++ compilé par CMake, la gestion des types de données et des erreurs, ainsi que l’utilisation des logs natifs. C’est une base solide pour des projets avancés comme le chiffrement natif, la détection anti-debug, le traitement d’image ou l’IA embarquée.

Conclusion

JNI est une passerelle puissante entre Java et C++, mais son usage doit être méthodique. Android le recommande pour des traitements réellement utiles : calcul intensif, réutilisation de bibliothèques natives existantes, logique sensible ou services bas niveau. Avec ce laboratoire, l’application JNIDemo devient un excellent point de départ pour des projets plus avancés (chiffrement natif, anti-debug, OpenCV, IA embarquée, sécurité applicative).
