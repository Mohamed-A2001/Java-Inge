# HashSet et HashMap en Java : Guide Complet

## Table des matières
1. [Introduction aux tables de hachage](#introduction)
2. [Principe de fonctionnement](#principe)
3. [HashSet](#hashset)
4. [HashMap](#hashmap)
5. [Gestion des collisions](#collisions)
6. [Bonnes pratiques](#bonnes-pratiques)

## Introduction aux tables de hachage <a name="introduction"></a>

Les tables de hachage sont des structures de données qui permettent de stocker et de récupérer des éléments très rapidement. En Java, elles sont implémentées principalement via deux classes :
- `HashSet` : pour stocker un ensemble de valeurs uniques
- `HashMap` : pour stocker des paires clé-valeur

Le principe fondamental repose sur une fonction de hachage qui convertit les données en une valeur numérique servant d'index dans un tableau.

## Principe de fonctionnement <a name="principe"></a>

### La fonction de hachage

```java
public class Example {
    @Override
    public int hashCode() {
        // Exemple simple de fonction de hachage pour une chaîne
        String str = this.toString();
        int hash = 0;
        for (int i = 0; i < str.length(); i++) {
            hash += str.charAt(i);
        }
        return hash;
    }
}
```

### Calcul de l'index

```java
// Index = hashCode % taille_tableau
int index = Math.abs(element.hashCode()) % arraySize;
```

## HashSet <a name="hashset"></a>

HashSet est une implémentation de l'interface Set qui n'accepte pas les doublons.

### Exemple complet d'utilisation

```java
import java.util.HashSet;

public class HashSetExample {
    public static void main(String[] args) {
        // Création d'un HashSet
        HashSet<String> fruits = new HashSet<>();

        // Ajout d'éléments
        fruits.add("Pomme");
        fruits.add("Banane");
        fruits.add("Orange");
        fruits.add("Pomme");  // Ne sera pas ajouté car doublon

        // Affichage
        System.out.println("Fruits: " + fruits);  // Ordre non garanti
        
        // Vérification de présence
        System.out.println("Contient Pomme? " + fruits.contains("Pomme")); // true
        
        // Suppression
        fruits.remove("Banane");
        
        // Parcours
        for (String fruit : fruits) {
            System.out.println(fruit);
        }
    }
}
```

## HashMap <a name="hashmap"></a>

HashMap permet de stocker des paires clé-valeur avec un accès rapide par clé.

### Exemple d'implémentation

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapExample {
    public static void main(String[] args) {
        // Création d'une HashMap
        HashMap<String, Integer> ages = new HashMap<>();

        // Ajout d'éléments
        ages.put("Alice", 25);
        ages.put("Bob", 30);
        ages.put("Charlie", 35);

        // Modification d'une valeur
        ages.put("Alice", 26);  // Écrase l'ancienne valeur

        // Récupération d'une valeur
        System.out.println("Age de Bob: " + ages.get("Bob")); // 30

        // Parcours des entrées
        for (Map.Entry<String, Integer> entry : ages.entrySet()) {
            System.out.println(entry.getKey() + " a " + entry.getValue() + " ans");
        }
        
        // Vérification de présence
        if (ages.containsKey("Alice")) {
            System.out.println("Age d'Alice trouvé !");
        }
    }
}
```

## Gestion des collisions <a name="collisions"></a>

Quand deux éléments différents produisent le même hashcode ou le même index, on parle de collision.

### Exemple de collision

```java
public class CollisionExample {
    public static void main(String[] args) {
        // Ces deux chaînes peuvent produire le même hashcode
        String s1 = "FB";
        String s2 = "Ea";
        
        System.out.println("Hashcode de FB: " + s1.hashCode());
        System.out.println("Hashcode de Ea: " + s2.hashCode());
        
        // Solution : chaînage
        class HashNode<K, V> {
            K key;
            V value;
            HashNode<K, V> next;
            
            HashNode(K key, V value) {
                this.key = key;
                this.value = value;
            }
        }
    }
}
```

## Bonnes pratiques <a name="bonnes-pratiques"></a>

### 1. Implémentation correcte de hashCode() et equals()

```java
public class Person {
    private String name;
    private int age;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

### 2. Choix de la capacité initiale

```java
// Pour un nombre connu d'éléments, initialiser avec une capacité appropriée
int expectedElements = 1000;
float loadFactor = 0.75f;
int capacity = (int) (expectedElements / loadFactor);

HashSet<String> optimizedSet = new HashSet<>(capacity);
HashMap<String, Integer> optimizedMap = new HashMap<>(capacity);
```

### 3. Performance et complexité

- Opérations moyennes en O(1) : add, remove, contains
- Pire cas en O(n) si beaucoup de collisions
- Le facteur de charge (load factor) par défaut est 0.75
- Redimensionnement automatique quand : taille > capacité × facteur de charge

### Exemple de benchmark simple

```java
public class HashPerformance {
    public static void main(String[] args) {
        HashSet<Integer> set = new HashSet<>();
        long start = System.nanoTime();
        
        // Test d'insertion
        for (int i = 0; i < 100000; i++) {
            set.add(i);
        }
        
        // Test de recherche
        for (int i = 0; i < 100000; i++) {
            set.contains(i);
        }
        
        long end = System.nanoTime();
        System.out.println("Temps total : " + (end - start) / 1_000_000 + " ms");
    }
}
```

## Points clés à retenir

1. Les HashSet et HashMap offrent des performances moyennes en O(1)
2. L'implémentation correcte de hashCode() et equals() est cruciale
3. Les collisions sont gérées automatiquement mais peuvent impacter les performances
4. L'ordre des éléments n'est pas garanti
5. Une bonne fonction de hachage doit répartir uniformément les valeurs
