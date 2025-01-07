# Les Classes et Objets en Java - Notes détaillées

## Table des matières
1. [Introduction aux objets](#1-introduction-aux-objets)
2. [Documentation et interface](#2-documentation-et-interface)
3. [Structure d'une classe](#3-structure-dune-classe)
4. [Les constructeurs](#4-les-constructeurs)
5. [Variables et méthodes d'instance](#5-variables-et-méthodes-dinstance)
6. [Encapsulation](#6-encapsulation)
7. [Composition et délégation](#7-composition-et-délégation)
8. [Bonnes pratiques](#8-bonnes-pratiques)

## 1. Introduction aux objets

Un objet en Java est une instance d'une classe qui encapsule :
- Des données (variables d'instance)
- Des comportements (méthodes)

Les objets permettent de :
- Structurer les données de manière cohérente
- Regrouper les données avec les opérations qui les manipulent
- Créer des abstractions réutilisables

## 2. Documentation et interface

### 2.1 Documentation d'une classe

La documentation d'une classe doit expliquer :
- Ce que représente un objet de la classe
- Comment créer un objet de cette classe 
- La liste des méthodes disponibles et leur comportement

### 2.2 Javadoc

La Javadoc est le format standard de documentation en Java :

```java
/**
 * Représente un compte bancaire.
 * Chaque compte a un titulaire, un numéro unique et un solde.
 */
public class Compte {
    // ...
}

/**
 * Effectue un dépôt sur le compte.
 * @param montant le montant à déposer (doit être positif)
 * @throws IllegalArgumentException si le montant est négatif
 */
public void depot(double montant) {
    // ...
}
```

## 3. Structure d'une classe

### 3.1 Déclaration de base

```java
public class Compte {
    // Variables d'instance (état)
    private int numero;
    private String titulaire;
    private double solde;
    
    // Constructeur(s)
    public Compte(int numero, String titulaire, double soldeInitial) {
        this.numero = numero;
        this.titulaire = titulaire;
        this.solde = soldeInitial;
    }
    
    // Méthodes (comportement)
    public void depot(double montant) {
        if (montant <= 0) {
            throw new IllegalArgumentException("Le montant doit être positif");
        }
        this.solde += montant;
    }
}
```

### 3.2 Variables d'instance

- Représentent l'état de l'objet
- Généralement déclarées private (encapsulation)
- Initialisées dans le constructeur
- Accessibles via des getters/setters

## 4. Les constructeurs

### 4.1 Constructeur par défaut

```java
public class Compte {
    private int numero;
    private double solde;
    
    // Constructeur par défaut
    public Compte() {
        this.numero = 0;
        this.solde = 0.0;
    }
}
```

### 4.2 Constructeurs paramétrés

```java
public class Compte {
    private int numero;
    private String titulaire;
    private double solde;
    
    // Constructeur complet
    public Compte(int numero, String titulaire, double soldeInitial) {
        this.numero = numero;
        this.titulaire = titulaire;
        this.solde = soldeInitial;
    }
    
    // Surcharge du constructeur
    public Compte(int numero, String titulaire) {
        this(numero, titulaire, 0.0); // Appel au constructeur complet
    }
}
```

## 5. Variables et méthodes d'instance

### 5.1 Variables d'instance

```java
public class Compte {
    // Variables d'instance
    private int numero;         // Stocke le numéro du compte
    private String titulaire;   // Stocke le nom du titulaire
    private double solde;       // Stocke le solde actuel
}
```

### 5.2 Méthodes d'instance

```java
public class Compte {
    // ... variables d'instance ...
    
    // Méthode modifiant l'état (mutateur)
    public void depot(double montant) {
        if (montant > 0) {
            this.solde += montant;
        }
    }
    
    // Méthode de consultation (accesseur)
    public double getSolde() {
        return this.solde;
    }
    
    // Méthode avec logique métier
    public boolean peutEffectuerRetrait(double montant) {
        return this.solde >= montant;
    }
}
```

## 6. Encapsulation

L'encapsulation est un principe fondamental de la POO qui consiste à :
- Cacher les détails d'implémentation
- Protéger l'état interne de l'objet
- Contrôler l'accès aux données

### 6.1 Exemple d'encapsulation

```java
public class Compte {
    private double solde;  // Variable privée
    
    // Getter public
    public double getSolde() {
        return this.solde;
    }
    
    // Méthode publique contrôlant la modification
    public void depot(double montant) {
        if (montant <= 0) {
            throw new IllegalArgumentException("Montant invalide");
        }
        this.solde += montant;
    }
    
    public boolean retrait(double montant) {
        if (montant <= 0) {
            throw new IllegalArgumentException("Montant invalide");
        }
        if (this.solde < montant) {
            return false;  // Retrait impossible
        }
        this.solde -= montant;
        return true;
    }
}
```

## 7. Composition et délégation

La composition permet de créer des objets complexes en combinant des objets plus simples.

### 7.1 Exemple de composition

```java
public class Banque {
    private String nom;
    private List<Compte> comptes;
    
    public Banque(String nom) {
        this.nom = nom;
        this.comptes = new ArrayList<>();
    }
    
    // Délégation : la méthode délègue l'opération au compte concerné
    public boolean effectuerRetrait(int numeroCompte, double montant) {
        Compte compte = trouverCompte(numeroCompte);
        if (compte != null) {
            return compte.retrait(montant);  // Délégation
        }
        return false;
    }
    
    private Compte trouverCompte(int numero) {
        return comptes.stream()
            .filter(c -> c.getNumero() == numero)
            .findFirst()
            .orElse(null);
    }
}
```

## 8. Bonnes pratiques

### 8.1 Principe d'encapsulation
- Déclarer les variables d'instance en private
- Fournir des méthodes d'accès contrôlées
- Valider les données avant modification

### 8.2 Initialisation cohérente
- Toujours initialiser complètement les objets via les constructeurs
- Vérifier la cohérence des données à l'initialisation
- Utiliser des constructeurs surchargés pour différents cas d'usage

### 8.3 Méthode toString()
```java
public class Compte {
    @Override
    public String toString() {
        return String.format("Compte{numero=%d, titulaire='%s', solde=%.2f}",
            numero, titulaire, solde);
    }
}
```

### 8.4 Documentation claire
- Documenter le rôle de chaque classe
- Documenter les préconditions et postconditions des méthodes
- Documenter les exceptions possibles

### 8.5 Exemple complet de classe bien conçue

```java
/**
 * Représente un compte bancaire avec les opérations de base.
 * Chaque compte a un titulaire unique et un solde qui ne peut pas être négatif.
 */
public class Compte {
    private final int numero;
    private final String titulaire;
    private double solde;
    private static int dernierNumero = 0;
    
    /**
     * Crée un nouveau compte.
     * @param titulaire le nom du titulaire (non null)
     * @param soldeInitial le solde initial (doit être positif ou nul)
     * @throws IllegalArgumentException si le titulaire est null ou le solde négatif
     */
    public Compte(String titulaire, double soldeInitial) {
        if (titulaire == null) {
            throw new IllegalArgumentException("Le titulaire ne peut pas être null");
        }
        if (soldeInitial < 0) {
            throw new IllegalArgumentException("Le solde initial doit être positif");
        }
        
        this.numero = ++dernierNumero;
        this.titulaire = titulaire;
        this.solde = soldeInitial;
    }
    
    public int getNumero() {
        return numero;
    }
    
    public String getTitulaire() {
        return titulaire;
    }
    
    public double getSolde() {
        return solde;
    }
    
    /**
     * Effectue un dépôt sur le compte.
     * @param montant le montant à déposer (doit être positif)
     * @throws IllegalArgumentException si le montant est négatif ou nul
     */
    public void depot(double montant) {
        if (montant <= 0) {
            throw new IllegalArgumentException("Le montant du dépôt doit être positif");
        }
        this.solde += montant;
    }
    
    /**
     * Effectue un retrait si le solde le permet.
     * @param montant le montant à retirer (doit être positif)
     * @return true si le retrait a été effectué, false sinon
     * @throws IllegalArgumentException si le montant est négatif ou nul
     */
    public boolean retrait(double montant) {
        if (montant <= 0) {
            throw new IllegalArgumentException("Le montant du retrait doit être positif");
        }
        if (solde < montant) {
            return false;
        }
        this.solde -= montant;
        return true;
    }
    
    @Override
    public String toString() {
        return String.format("Compte n°%d - Titulaire: %s - Solde: %.2f€",
            numero, titulaire, solde);
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Compte compte = (Compte) o;
        return numero == compte.numero;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(numero);
    }
}
```

Ce document fournit une base solide pour comprendre et implémenter la programmation orientée objet en Java. Les concepts et exemples présentés peuvent être adaptés et étendus selon les besoins spécifiques de chaque projet.
