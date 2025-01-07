# Guide Complet : Spécification et Test des Classes Java

## Table des matières
1. [Introduction](#introduction)
2. [Spécification des Classes](#spécification-des-classes)
3. [Contrats et Invariants](#contrats-et-invariants)
4. [Tests Unitaires](#tests-unitaires)
5. [Bonnes Pratiques](#bonnes-pratiques)
6. [Exemples Complets](#exemples-complets)

## Introduction

La spécification et le test des classes sont des aspects fondamentaux du développement logiciel orienté objet. Ce guide couvre les principes essentiels et les meilleures pratiques pour Java.

## Spécification des Classes

### Formes de Spécification

Une spécification de classe peut prendre deux formes :
1. Interface Java + contrats JavaDoc
2. Squelette de classe + contrats JavaDoc

### Éléments à Spécifier

Deux aspects principaux doivent être spécifiés :
1. L'état interne de l'objet et ses conditions de cohérence
2. Les contrats des méthodes (préconditions, postconditions)

## Contrats et Invariants

### Invariant de Classe

L'invariant définit les conditions que l'état interne d'un objet doit toujours respecter :
- À la création de l'objet
- Avant et après chaque appel de méthode
- Tout au long du cycle de vie de l'objet

Exemple d'invariant :
```java
/**
 * Représente un compte bancaire avec solde toujours positif.
 * @invariant getSolde() > 0
 */
public interface CompteBancaire {
    double getSolde();
    void depot(double montant);
    void retrait(double montant);
}
```

### Contrats des Méthodes

Chaque méthode doit spécifier :
- Préconditions (conditions requises avant l'exécution)
- Postconditions (garanties après l'exécution)
- Exceptions possibles

Exemple de contrat de méthode :
```java
/**
 * Effectue un retrait sur le compte.
 * @param montant Le montant à retirer
 * @pre montant > 0 && getSolde() >= montant
 * @post getSolde() == old(getSolde()) - montant
 * @throws IllegalArgumentException si le montant est négatif ou supérieur au solde
 */
void retrait(double montant);
```

## Tests Unitaires

### Structure des Tests

Les tests doivent vérifier :
1. La cohérence de l'état interne
2. Le respect des contrats des méthodes
3. La gestion correcte des cas d'erreur

### Exemple Complet de Tests

```java
public class CompteBancaireTest {
    private CompteBancaire compte;
    
    @BeforeEach
    void setUp() {
        compte = new CompteBancaireImpl(100.0); // Solde initial de 100
    }
    
    @Test
    void testConstructeurValide() {
        assertDoesNotThrow(() -> new CompteBancaireImpl(50.0));
        assertEquals(100.0, compte.getSolde());
    }
    
    @Test
    void testConstructeurInvalide() {
        assertThrows(IllegalArgumentException.class, 
            () -> new CompteBancaireImpl(-50.0));
    }
    
    @Test
    void testRetraitValide() {
        double soldePrecedent = compte.getSolde();
        compte.retrait(50.0);
        assertEquals(soldePrecedent - 50.0, compte.getSolde());
        assertTrue(compte.getSolde() > 0);
    }
    
    @Test
    void testRetraitInvalide() {
        assertThrows(IllegalArgumentException.class, 
            () -> compte.retrait(150.0));
    }
}
```

## Bonnes Pratiques

### Encapsulation et Tests

1. Protégez l'état interne (variables private/protected)
2. Fournissez des méthodes d'accès (getters) pour les tests
3. Implémentez une méthode `invariantOK()` pour vérifier l'invariant

```java
public class CompteBancaireImpl implements CompteBancaire {
    private double solde;
    
    protected boolean invariantOK() {
        return solde > 0;
    }
    
    // Vérification systématique après chaque opération
    private void verifierInvariant() {
        if (!invariantOK()) {
            throw new IllegalStateException("Invariant violé");
        }
    }
}
```

### Organisation des Tests

1. Tests positifs (cas normaux)
2. Tests négatifs (cas d'erreur)
3. Tests des cas limites
4. Tests de l'invariant

## Exemples Complets

### Exemple : Pile Bornée

```java
/**
 * Représente une pile d'éléments de taille limitée.
 * @invariant getTaille() >= 0 && getTaille() <= getCapaciteMax()
 */
public interface PileBornee<T> {
    /**
     * Empile un élément.
     * @param element L'élément à empiler
     * @pre !estPleine()
     * @post getTaille() == old(getTaille()) + 1
     * @throws IllegalStateException si la pile est pleine
     */
    void empiler(T element);
    
    /**
     * Dépile un élément.
     * @return L'élément au sommet de la pile
     * @pre !estVide()
     * @post getTaille() == old(getTaille()) - 1
     * @throws IllegalStateException si la pile est vide
     */
    T depiler();
    
    int getTaille();
    int getCapaciteMax();
    boolean estVide();
    boolean estPleine();
}
```

Implémentation :
```java
public class PileBorneeImpl<T> implements PileBornee<T> {
    private final T[] elements;
    private int taille;
    private final int capaciteMax;
    
    @SuppressWarnings("unchecked")
    public PileBorneeImpl(int capacite) {
        if (capacite <= 0) {
            throw new IllegalArgumentException("La capacité doit être positive");
        }
        this.capaciteMax = capacite;
        this.elements = (T[]) new Object[capacite];
        this.taille = 0;
    }
    
    protected boolean invariantOK() {
        return taille >= 0 && taille <= capaciteMax;
    }
    
    @Override
    public void empiler(T element) {
        if (estPleine()) {
            throw new IllegalStateException("Pile pleine");
        }
        elements[taille++] = element;
        assert invariantOK() : "Invariant violé après empiler";
    }
    
    @Override
    public T depiler() {
        if (estVide()) {
            throw new IllegalStateException("Pile vide");
        }
        T element = elements[--taille];
        elements[taille] = null; // Aide au garbage collector
        assert invariantOK() : "Invariant violé après depiler";
        return element;
    }
    
    // ... autres méthodes
}
```

Tests :
```java
public class PileBorneeTest {
    private PileBornee<String> pile;
    
    @BeforeEach
    void setUp() {
        pile = new PileBorneeImpl<>(3);
    }
    
    @Test
    void testConstructeurValide() {
        assertDoesNotThrow(() -> new PileBorneeImpl<String>(5));
    }
    
    @Test
    void testConstructeurInvalide() {
        assertThrows(IllegalArgumentException.class, 
            () -> new PileBorneeImpl<String>(0));
    }
    
    @Test
    void testEmpilerDepiler() {
        pile.empiler("A");
        assertEquals(1, pile.getTaille());
        assertEquals("A", pile.depiler());
        assertEquals(0, pile.getTaille());
    }
    
    @Test
    void testPilePleine() {
        pile.empiler("A");
        pile.empiler("B");
        pile.empiler("C");
        assertThrows(IllegalStateException.class, 
            () -> pile.empiler("D"));
    }
    
    @Test
    void testPileVide() {
        assertThrows(IllegalStateException.class, 
            () -> pile.depiler());
    }
}
```

Ce guide couvre les aspects essentiels de la spécification et du test des classes en Java. Pour approfondir certains aspects ou avoir plus d'exemples, n'hésitez pas à consulter la documentation officielle de JUnit et les bonnes pratiques de test unitaire.
