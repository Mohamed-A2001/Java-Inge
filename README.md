# Les Collections Triées en Java - Guide Complet

## Table des matières
1. [Introduction](#introduction)
2. [Collections triées par construction](#collections-triées-par-construction)
3. [Interfaces de comparaison](#interfaces-de-comparaison)
4. [Exemple complet avec Comparable](#exemple-complet-avec-comparable)
5. [Exemple complet avec Comparator](#exemple-complet-avec-comparator)
6. [Utilisation de Collections.sort](#utilisation-de-collectionssort)
7. [Bonnes pratiques et pièges à éviter](#bonnes-pratiques-et-pièges-à-éviter)

## Introduction

Les collections triées en Java permettent de maintenir des éléments dans un ordre spécifique. Il existe deux approches principales :

1. Collections triées par construction (SortedSet, NavigableSet, SortedMap)
2. Tri a posteriori avec Collections.sort()

## Collections triées par construction

### SortedSet<E>
Interface principale pour les ensembles triés. Caractéristiques :
- Pas de doublons
- Éléments toujours triés
- Implantation principale : TreeSet<E>

```java
// Exemple de SortedSet
SortedSet<String> noms = new TreeSet<>();
noms.add("Pierre");
noms.add("Alice");
noms.add("Bernard");
// Affichera : Alice, Bernard, Pierre
```

### NavigableSet<E>
Extension de SortedSet avec des opérations de navigation supplémentaires :
```java
NavigableSet<Integer> numbers = new TreeSet<>();
numbers.add(1);
numbers.add(2);
numbers.add(5);
numbers.add(10);

// Trouver l'élément le plus proche inférieur à 6
Integer lower = numbers.lower(6); // retourne 5
// Trouver l'élément le plus proche supérieur à 6
Integer higher = numbers.higher(6); // retourne 10
```

### SortedMap<K,V>
Dictionnaire avec clés triées :
```java
SortedMap<String, Integer> scores = new TreeMap<>();
scores.put("Alice", 95);
scores.put("Bob", 85);
scores.put("Charlie", 90);

// Obtenir une sous-map des scores commençant par A à C
SortedMap<String, Integer> subMap = scores.subMap("A", "D");
```

## Interfaces de comparaison

### Comparable<E>
Interface pour définir l'ordre naturel d'une classe :

```java
public class Personne implements Comparable<Personne> {
    private String nom;
    private int age;
    
    public Personne(String nom, int age) {
        this.nom = nom;
        this.age = age;
    }
    
    @Override
    public int compareTo(Personne autre) {
        // Tri par nom
        return this.nom.compareTo(autre.nom);
    }
    
    // N'oubliez pas equals et hashCode!
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Personne)) return false;
        Personne autre = (Personne) obj;
        return this.nom.equals(autre.nom) && this.age == autre.age;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(nom, age);
    }
}
```

### Comparator<E>
Interface pour définir des ordres de tri alternatifs :

```java
// Comparateur pour trier les personnes par âge
public class ComparateurAge implements Comparator<Personne> {
    @Override
    public int compare(Personne p1, Personne p2) {
        return Integer.compare(p1.getAge(), p2.getAge());
    }
}

// Utilisation avec un TreeSet
SortedSet<Personne> personnesParAge = new TreeSet<>(new ComparateurAge());
```

## Exemple complet avec Comparable

Voici un exemple complet d'une classe Article implémentant Comparable :

```java
public class Article implements Comparable<Article> {
    private String reference;
    private String nom;
    private double prix;
    
    public Article(String reference, String nom, double prix) {
        this.reference = reference;
        this.nom = nom;
        this.prix = prix;
    }
    
    // Getters
    public String getReference() { return reference; }
    public String getNom() { return nom; }
    public double getPrix() { return prix; }
    
    @Override
    public int compareTo(Article autre) {
        // Tri naturel par référence
        return this.reference.compareTo(autre.reference);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Article)) return false;
        Article autre = (Article) obj;
        return Objects.equals(reference, autre.reference) &&
               Objects.equals(nom, autre.nom) &&
               Double.compare(prix, autre.prix) == 0;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(reference, nom, prix);
    }
    
    @Override
    public String toString() {
        return String.format("Article[ref=%s, nom=%s, prix=%.2f€]", 
                           reference, nom, prix);
    }
}
```

## Exemple complet avec Comparator

Plusieurs comparateurs pour la classe Article :

```java
// Comparateur par prix
public class ComparateurPrix implements Comparator<Article> {
    @Override
    public int compare(Article a1, Article a2) {
        return Double.compare(a1.getPrix(), a2.getPrix());
    }
}

// Comparateur par nom
public class ComparateurNom implements Comparator<Article> {
    @Override
    public int compare(Article a1, Article a2) {
        return a1.getNom().compareTo(a2.getNom());
    }
}

// Utilisation
public class ExempleComparateurs {
    public static void main(String[] args) {
        // Collection d'articles
        List<Article> articles = Arrays.asList(
            new Article("REF001", "Ordinateur", 999.99),
            new Article("REF002", "Souris", 29.99),
            new Article("REF003", "Clavier", 59.99)
        );
        
        // Tri par prix
        TreeSet<Article> articlesParPrix = new TreeSet<>(new ComparateurPrix());
        articlesParPrix.addAll(articles);
        
        // Tri par nom
        TreeSet<Article> articlesParNom = new TreeSet<>(new ComparateurNom());
        articlesParNom.addAll(articles);
        
        // Affichage des deux tris
        System.out.println("Articles par prix croissant :");
        articlesParPrix.forEach(System.out::println);
        
        System.out.println("\nArticles par nom :");
        articlesParNom.forEach(System.out::println);
    }
}
```

## Utilisation de Collections.sort

La classe Collections fournit des méthodes statiques pour trier les List :

```java
public class ExempleCollectionsSort {
    public static void main(String[] args) {
        List<Article> articles = new ArrayList<>();
        articles.add(new Article("REF001", "Ordinateur", 999.99));
        articles.add(new Article("REF002", "Souris", 29.99));
        articles.add(new Article("REF003", "Clavier", 59.99));
        
        // Tri naturel (utilise compareTo)
        Collections.sort(articles);
        
        // Tri avec un comparateur spécifique
        Collections.sort(articles, new ComparateurPrix());
        
        // Utilisation de Comparator.comparing (Java 8+)
        Collections.sort(articles, Comparator.comparing(Article::getPrix));
        
        // Tri inversé
        Collections.sort(articles, Comparator.comparing(Article::getPrix).reversed());
        
        // Tri multiple
        Collections.sort(articles, 
            Comparator.comparing(Article::getPrix)
                     .thenComparing(Article::getNom));
    }
}
```

## Bonnes pratiques et pièges à éviter

1. **Cohérence equals/compareTo** :
   - Si x.equals(y) est vrai, alors x.compareTo(y) doit retourner 0
   - Si x.compareTo(y) == 0, alors x.equals(y) doit être vrai

2. **Comparaison null** :
   - Toujours vérifier les null dans les comparateurs
   ```java
   public int compare(Article a1, Article a2) {
       if (a1 == a2) return 0;
       if (a1 == null) return -1;
       if (a2 == null) return 1;
       return a1.getNom().compareTo(a2.getNom());
   }
   ```

3. **Performance** :
   - TreeSet/TreeMap : O(log n) pour add/remove/contains
   - ArrayList.sort() : O(n log n)
   - Préférer TreeSet si beaucoup d'insertions/suppressions triées

4. **Immutabilité** :
   - Ne pas modifier les champs utilisés pour la comparaison après insertion dans une collection triée

### Exemple de classe robuste et complète

```java
public class Produit implements Comparable<Produit> {
    private final String code;  // final pour l'immutabilité
    private final String designation;
    private final BigDecimal prix;
    
    public Produit(String code, String designation, BigDecimal prix) {
        this.code = Objects.requireNonNull(code, "Le code ne peut être null");
        this.designation = Objects.requireNonNull(designation, "La désignation ne peut être null");
        this.prix = Objects.requireNonNull(prix, "Le prix ne peut être null");
        if (prix.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Le prix ne peut être négatif");
        }
    }
    
    // Getters (pas de setters pour préserver l'immutabilité)
    public String getCode() { return code; }
    public String getDesignation() { return designation; }
    public BigDecimal getPrix() { return prix; }
    
    @Override
    public int compareTo(Produit autre) {
        if (autre == null) return 1;
        return this.code.compareTo(autre.code);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Produit)) return false;
        Produit autre = (Produit) obj;
        return Objects.equals(code, autre.code) &&
               Objects.equals(designation, autre.designation) &&
               prix.compareTo(autre.prix) == 0;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(code, designation, prix);
    }
    
    @Override
    public String toString() {
        return String.format("Produit[code=%s, designation=%s, prix=%s€]",
                           code, designation, prix.toString());
    }
}
```

Ce guide couvre les aspects essentiels des collections triées en Java. Pour approfondir, consultez la documentation officielle de Java et explorez les nouvelles fonctionnalités disponibles dans les versions récentes de Java.
