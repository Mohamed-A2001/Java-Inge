# Java - Égalité d'objets et Collections

## Table des matières

1. [Introduction](#introduction)
2. [Le problème de base](#le-problème-de-base)
3. [Rôle de equals() et hashCode()](#rôle-de-equals-et-hashcode)
4. [Collections et tables de hachage](#collections-et-tables-de-hachage)
5. [Redéfinition des méthodes](#redéfinition-des-méthodes)

## Introduction

Dans Java, l'égalité entre objets est un concept fondamental, particulièrement important pour le bon fonctionnement des collections. Ce document détaille comment implémenter correctement l'égalité dans vos classes.

## Le problème de base

Prenons un exemple simple avec une classe `Heure` :

```java
public class Heure {
    private final int heure;
    private final int minutes;
    
    public Heure(int h, int m) {
        this.heure = h;
        this.minutes = m;
    }
    
    // getters...
    public int getHeure() { return heure; }
    public int getMinutes() { return minutes; }
}
```

Sans redéfinition de equals() et hashCode(), nous rencontrons des problèmes :

```java
// Création de deux objets avec les mêmes valeurs
Heure h1 = new Heure(13, 45);
Heure h2 = new Heure(13, 45);

// Test d'égalité
System.out.println(h1.equals(h2)); // Affiche false !

// Problème avec les collections
List<Heure> liste = new ArrayList<>();
liste.add(h1);
System.out.println(liste.contains(h2)); // Affiche false !

Set<Heure> ensemble = new HashSet<>();
ensemble.add(h1);
ensemble.add(h2); // Ajoute un doublon !
System.out.println(ensemble.size()); // Affiche 2 au lieu de 1 !
```

## Rôle de equals() et hashCode()

### equals()

La méthode equals() doit respecter les propriétés suivantes :
- Réflexivité : x.equals(x) doit renvoyer true
- Symétrie : si x.equals(y) est true, alors y.equals(x) doit être true
- Transitivité : si x.equals(y) et y.equals(z), alors x.equals(z)
- Cohérence avec hashCode()

### hashCode()

La méthode hashCode() doit :
- Être cohérente avec equals() : si deux objets sont égaux selon equals(), ils doivent avoir le même hashCode()
- Retourner le même hashCode pour un objet non modifié
- Distribuer efficacement les objets dans une table de hachage

## Collections et tables de hachage

Les collections utilisent ces méthodes en interne :

```java
// Exemple avec HashSet
public class HashSetExample {
    public static void main(String[] args) {
        Set<Heure> horaires = new HashSet<>();
        
        // Sans redéfinition correcte :
        Heure h1 = new Heure(9, 0);
        Heure h2 = new Heure(9, 0);
        
        horaires.add(h1);
        horaires.add(h2); // Ne devrait pas ajouter de doublon
        
        // Devrait afficher 1, mais affiche 2 sans redéfinition !
        System.out.println("Taille : " + horaires.size());
    }
}
```

### Fonctionnement interne d'une table de hachage

```java
// Pseudo-code du fonctionnement
int index = Math.abs(object.hashCode()) % tableSize;
LinkedList<E> bucket = table[index];
for (E element : bucket) {
    if (element.equals(object)) {
        return true; // Élément trouvé
    }
}
```

## Redéfinition des méthodes

### Exemple complet avec la classe Heure

```java
public class Heure {
    private final int heure;
    private final int minutes;
    
    public Heure(int h, int m) {
        this.heure = h;
        this.minutes = m;
    }
    
    @Override
    public boolean equals(Object obj) {
        // 1. Test référence
        if (this == obj) return true;
        
        // 2. Test null
        if (obj == null) return false;
        
        // 3. Test classe
        if (getClass() != obj.getClass()) return false;
        
        // 4. Cast
        Heure other = (Heure) obj;
        
        // 5. Comparaison des champs
        return this.heure == other.heure && 
               this.minutes == other.minutes;
    }
    
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + heure;
        result = prime * result + minutes;
        return result;
    }
    
    // Version alternative avec Objects.hash
    @Override
    public int hashCode() {
        return Objects.hash(heure, minutes);
    }
}
```

### Exemple avec une classe plus complexe

```java
public class Personne {
    private final String nom;
    private final String prenom;
    private final LocalDate dateNaissance;
    private final List<String> adresses;
    
    public Personne(String nom, String prenom, LocalDate dateNaissance,
                    List<String> adresses) {
        this.nom = nom;
        this.prenom = prenom;
        this.dateNaissance = dateNaissance;
        this.adresses = new ArrayList<>(adresses); // Copie défensive
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        Personne other = (Personne) obj;
        return Objects.equals(nom, other.nom) &&
               Objects.equals(prenom, other.prenom) &&
               Objects.equals(dateNaissance, other.dateNaissance) &&
               Objects.equals(adresses, other.adresses);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(nom, prenom, dateNaissance, adresses);
    }
}
```

### Test complet

```java
public class TestEgalite {
    public static void main(String[] args) {
        // Test avec Heure
        Heure h1 = new Heure(13, 45);
        Heure h2 = new Heure(13, 45);
        Heure h3 = new Heure(14, 30);
        
        // Tests basiques
        System.out.println("h1 equals h2: " + h1.equals(h2)); // true
        System.out.println("h1 equals h3: " + h1.equals(h3)); // false
        
        // Test avec Set
        Set<Heure> heures = new HashSet<>();
        heures.add(h1);
        heures.add(h2);
        System.out.println("Taille Set: " + heures.size()); // 1
        
        // Test avec Map
        Map<Heure, String> agenda = new HashMap<>();
        agenda.put(h1, "Réunion");
        System.out.println("Agenda h2: " + agenda.get(h2)); // "Réunion"
        
        // Test avec List
        List<Heure> liste = new ArrayList<>();
        liste.add(h1);
        System.out.println("Contains h2: " + liste.contains(h2)); // true
    }
}
```

## Bonnes pratiques

1. **Toujours redéfinir equals() et hashCode() ensemble**
   - Ne jamais redéfinir l'un sans l'autre
   - Utiliser les mêmes champs dans les deux méthodes

2. **Utiliser les outils disponibles**
   - IDE : génération automatique des méthodes
   - Objects.equals() pour la comparaison sûre
   - Objects.hash() pour le calcul du hashCode

3. **Attention aux collections mutables**
   - Faire des copies défensives
   - Ne pas utiliser de champs mutables dans hashCode()

4. **Tests**
   - Tester la réflexivité, symétrie et transitivité
   - Tester le comportement avec différentes collections
   - Vérifier les performances avec de grands ensembles de données

## Conclusion

Une bonne implémentation de equals() et hashCode() est cruciale pour :
- Le bon fonctionnement des collections
- La cohérence du comportement de l'application
- Les performances des opérations de recherche

N'hésitez pas à utiliser les outils de génération automatique de votre IDE, mais comprendre les principes sous-jacents reste essentiel pour déboguer et optimiser votre code.
