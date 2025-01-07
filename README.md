# Héritage et Polymorphisme en Java

## 1. Extension de types objet en Java

### 1.1 Les trois types d'objets en Java

Java propose trois sortes de types pour les objets :
- Interfaces
- Classes
- Classes abstraites

Ces trois types sont extensibles via différents mécanismes :

- `interface extends interface` : ajout de profils de méthodes
- `classe implements interface` : ajout de variables / implémentation / méthodes  
- `classe extends classe` : ajout de méthodes/variables

Le nouveau type "hérite" des membres du type existant et peut en introduire d'autres.

À noter : Une classe sans `extends` étend implicitement la classe `Object`.

### 1.2 Exemple avec les interfaces de régimes alimentaires

```java
public interface Carnivore {
    void manger(Animal a);
}

public interface Herbivore {
    void manger(Vegetal v); 
}

public interface Omnivore extends Carnivore, Herbivore {
    double getRapportViandeVege();
}

public class Humain extends Animal implements Omnivore {
    @Override
    public void manger(Animal animal) {
        // Implémentation
    }
    
    @Override
    public void manger(Vegetal vegetal) {
        // Implémentation
    }
    
    @Override
    public double getRapportViandeVege() {
        // Implémentation
        return ratio;
    }
}
```

## 2. Hiérarchie et compatibilité

### 2.1 La relation "est-un" 

Le type objet B "est-un" A si B est obtenu à partir de A via héritage (`extends`) et/ou implémentation (`implements`) successifs.

Exemples :
- Humain est un Animal, Object, Omnivore, Carnivore...
- Omnivore est un Carnivore et un Herbivore

Important : l'inverse n'est pas vrai (Animal n'est pas forcément un Humain)

### 2.2 Compatibilité ascendante

La compatibilité ascendante permet qu'une instance d'une sous-classe soit utilisée partout où le type parent est attendu.

```java
Animal a = new Humain();      // OK car Humain est-un Animal
Omnivore o = new Humain();    // OK car Humain est-un Omnivore 
Object obj = new Humain();    // OK car toute classe est-un Object
```

## 3. Polymorphisme objet

### 3.1 Type déclaré vs Type constaté

Pour une variable :
- Type déclaré (statique) : celui donné à la déclaration, unique et ne change pas
- Type constaté (dynamique) : type réel de l'objet pointé à un moment donné, peut changer

Exemple :
```java
Deplacable x = new Point(1,2);     // Type statique: Deplacable, Type dynamique: Point
x = new Cercle(x, 3);              // Type statique: Deplacable, Type dynamique: Cercle
```

### 3.2 Polymorphisme et abstraction

Exemple avec une scène graphique :

```java
public interface Deplacable {
    int getX();
    int getY();
    void move(int dx, int dy);
}

public class Point implements Deplacable {
    private int x, y;
    
    @Override
    public void move(int dx, int dy) {
        this.x += dx;
        this.y += dy;
    }
    // autres implémentations
}

public class Scene {
    private ArrayList<Deplacable> scene;
    
    public void moveAll(int dx, int dy) {
        for(Deplacable fig : scene) {
            fig.move(dx, dy);  // Appel polymorphe
        }
    }
    
    public void add(Deplacable fig) {
        scene.add(fig);
    }
}
```

## 4. Héritage

### 4.1 Héritage et constructeurs

Les constructeurs ne sont pas hérités. La sous-classe doit explicitement appeler le constructeur parent via `super()` :

```java
public class CompteDecouvert extends Compte {
    private double decouvertMax;
    
    public CompteDecouvert(double soldeInitial, double decouvertMax) {
        super(soldeInitial);  // Appel constructeur parent
        this.decouvertMax = decouvertMax;
    }
}
```

### 4.2 Protected et visibilité

Le modificateur `protected` permet l'accès aux membres depuis les sous-classes :

```java
public class Compte {
    protected double solde;  // Accessible dans les sous-classes
    
    public void depot(double montant) {
        this.solde += montant;
    }
}

public class CompteEpargne extends Compte {
    private double taux;
    
    public void calculerInterets() {
        this.solde *= (1 + taux);  // OK car solde est protected
    }
}
```

### 4.3 La variable super

`super` permet d'accéder aux membres de la super-classe :

```java
public class CompteDecouvert extends Compte {
    @Override
    public void retrait(double montant) {
        if (solde - montant >= -decouvertMax) {
            super.retrait(montant);  // Appel méthode parent
        } else {
            throw new SoldeInsuffisantException();
        }
    }
}
```

## 5. Liaison dynamique

La liaison dynamique détermine quelle version d'une méthode est exécutée en fonction du type réel (dynamique) de l'objet :

```java
Compte c = new CompteDecouvert(1000, 500);
c.retrait(1200);  // Exécute la méthode de CompteDecouvert
```

C'est un mécanisme fondamental qui permet le polymorphisme et la flexibilité dans la conception orientée objet.

### Points clés à retenir :

1. Le type statique est vérifié à la compilation
2. Le type dynamique détermine le comportement à l'exécution
3. Les méthodes sont liées dynamiquement
4. L'héritage crée une relation "est-un"
5. protected permet l'accès aux sous-classes
6. super permet d'accéder à la super-classe
7. Les constructeurs doivent être explicitement chaînés
