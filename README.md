# Guide Complet des Maps en Java

## Table des matières
1. [Introduction aux Maps](#introduction)
2. [Structure et concepts de base](#structure)
3. [Les différentes implémentations](#implementations)
4. [Opérations de base](#operations)
5. [Les vues d'une Map](#vues)
6. [Exemples pratiques](#exemples)
7. [Contraintes et bonnes pratiques](#contraintes)

## Introduction aux Maps <a name="introduction"></a>

Une Map en Java est une structure de données qui permet de stocker des paires clé-valeur. Chaque élément dans une Map est une association entre une clé unique et une valeur. Les Maps sont particulièrement utiles lorsqu'on souhaite créer des dictionnaires, des caches, ou toute autre structure nécessitant un accès rapide aux données via une clé.

### Caractéristiques principales :
- Chaque clé est unique
- Une clé ne peut être associée qu'à une seule valeur
- Les valeurs peuvent être dupliquées
- L'ordre des éléments dépend de l'implémentation utilisée

## Structure et concepts de base <a name="structure"></a>

### La hiérarchie des Maps

```plain
Map (interface)
├── HashMap
├── LinkedHashMap
├── TreeMap
├── WeakHashMap
└── IdentityHashMap
```

### Déclaration d'une Map

```java
// Déclaration générique
Map<K,V> map;

// Implémentations spécifiques
Map<String, Integer> hashMap = new HashMap<>();
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
Map<String, Integer> treeMap = new TreeMap<>();
```

## Les différentes implémentations <a name="implementations"></a>

### HashMap
- Implémentation la plus courante
- Performance optimale pour les opérations de base (O(1) en moyenne)
- Pas d'ordre garanti des éléments

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 87);
scores.put("Charlie", 91);
```

### LinkedHashMap
- Maintient l'ordre d'insertion
- Légèrement plus lente que HashMap
- Utile quand l'ordre est important

```java
Map<String, String> capitals = new LinkedHashMap<>();
capitals.put("France", "Paris");
capitals.put("Italy", "Rome");
capitals.put("Spain", "Madrid");
// L'itération respectera l'ordre d'insertion
```

### TreeMap
- Maintient les clés triées selon leur ordre naturel ou un comparateur
- Performance O(log n) pour les opérations de base
- Utile pour les données qui doivent être maintenues triées

```java
TreeMap<Integer, String> ranking = new TreeMap<>();
ranking.put(3, "Bronze");
ranking.put(1, "Gold");
ranking.put(2, "Silver");
// L'itération sera toujours dans l'ordre: 1,2,3
```

## Opérations de base <a name="operations"></a>

### Ajout et modification d'éléments

```java
Map<String, Integer> ages = new HashMap<>();

// Ajout d'éléments
ages.put("Alice", 25);
ages.put("Bob", 30);

// Modification d'un élément
ages.put("Alice", 26);  // Écrase l'ancienne valeur

// Ajout conditionnel
ages.putIfAbsent("Charlie", 22);  // Ajoute seulement si la clé n'existe pas
```

### Récupération d'éléments

```java
Map<String, Integer> stock = new HashMap<>();
stock.put("Pommes", 50);

// Récupération simple
Integer quantity = stock.get("Pommes");  // Retourne 50

// Récupération avec valeur par défaut
Integer bananas = stock.getOrDefault("Bananes", 0);  // Retourne 0

// Vérification de présence
boolean hasPommes = stock.containsKey("Pommes");  // true
boolean has50 = stock.containsValue(50);  // true
```

### Suppression d'éléments

```java
Map<String, String> users = new HashMap<>();
users.put("admin", "root");

// Suppression simple
users.remove("admin");

// Suppression conditionnelle
users.remove("admin", "root");  // Supprime seulement si la valeur correspond

// Suppression de tous les éléments
users.clear();
```

## Les vues d'une Map <a name="vues"></a>

Les Maps offrent trois types de vues différentes :

### Vue des clés (KeySet)

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "One");
map.put(2, "Two");

Set<Integer> keys = map.keySet();
for (Integer key : keys) {
    System.out.println("Clé: " + key);
}
```

### Vue des valeurs (Values)

```java
Collection<String> values = map.values();
for (String value : values) {
    System.out.println("Valeur: " + value);
}
```

### Vue des entrées (EntrySet)

```java
Set<Map.Entry<Integer, String>> entries = map.entrySet();
for (Map.Entry<Integer, String> entry : entries) {
    System.out.println("Clé: " + entry.getKey() + ", Valeur: " + entry.getValue());
}
```

## Exemples pratiques <a name="exemples"></a>

### Exemple 1: Compteur de fréquence des mots

```java
public class WordFrequencyCounter {
    public static Map<String, Integer> countWords(String text) {
        Map<String, Integer> frequency = new HashMap<>();
        String[] words = text.toLowerCase().split("\\s+");
        
        for (String word : words) {
            frequency.merge(word, 1, Integer::sum);
        }
        
        return frequency;
    }

    public static void main(String[] args) {
        String text = "Java est un langage Java est populaire";
        Map<String, Integer> freq = countWords(text);
        System.out.println(freq);  // {java=2, est=2, un=1, langage=1, populaire=1}
    }
}
```

### Exemple 2: Cache simple avec LinkedHashMap

```java
public class SimpleCache<K,V> extends LinkedHashMap<K,V> {
    private final int capacity;
    
    public SimpleCache(int capacity) {
        super(capacity + 1, 1.1f, true);
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return size() > capacity;
    }

    public static void main(String[] args) {
        SimpleCache<String, String> cache = new SimpleCache<>(3);
        cache.put("A", "1");
        cache.put("B", "2");
        cache.put("C", "3");
        cache.put("D", "4");  // A sera automatiquement supprimé
        System.out.println(cache);  // {B=2, C=3, D=4}
    }
}
```

## Contraintes et bonnes pratiques <a name="contraintes"></a>

### Contraintes sur les clés

1. Pour HashMap :
   - La méthode `equals()` doit être correctement implémentée
   - La méthode `hashCode()` doit être cohérente avec `equals()`

```java
public class CustomKey {
    private final String id;
    private final int code;

    public CustomKey(String id, int code) {
        this.id = id;
        this.code = code;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        CustomKey customKey = (CustomKey) o;
        return code == customKey.code && Objects.equals(id, customKey.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, code);
    }
}
```

2. Pour TreeMap :
   - Les clés doivent implémenter Comparable, ou
   - Un Comparator doit être fourni au constructeur

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age);
    }
}

// Utilisation avec TreeMap
TreeMap<Person, String> people = new TreeMap<>();

// Ou avec un Comparator personnalisé
TreeMap<Person, String> peopleByName = new TreeMap<>((p1, p2) -> 
    p1.getName().compareTo(p2.getName())
);
```

### Bonnes pratiques

1. Choisir la bonne implémentation :
   - HashMap pour un usage général
   - LinkedHashMap quand l'ordre d'insertion est important
   - TreeMap quand les clés doivent être triées

2. Utiliser les méthodes de Java 8+ :
   - `computeIfAbsent()`
   - `computeIfPresent()`
   - `merge()`

```java
Map<String, List<String>> groups = new HashMap<>();

// Ancien style
if (!groups.containsKey("A")) {
    groups.put("A", new ArrayList<>());
}
groups.get("A").add("membre");

// Nouveau style (préféré)
groups.computeIfAbsent("A", k -> new ArrayList<>()).add("membre");
```

3. Utiliser les méthodes de sécurité :
   - `getOrDefault()` au lieu de vérifier `null`
   - `putIfAbsent()` pour les insertions conditionnelles

4. Considérer l'immutabilité :
   - Utiliser `Collections.unmodifiableMap()` pour créer des maps immuables
   - Utiliser `Map.of()` ou `Map.ofEntries()` pour de petites maps immuables (Java 9+)

```java
// Map immuable avec Java 9+
Map<String, Integer> constants = Map.of(
    "PI", 3,
    "E", 2,
    "PHI", 1
);

// Map immuable à partir d'une map existante
Map<String, Integer> immutableMap = Collections.unmodifiableMap(new HashMap<>());
```

