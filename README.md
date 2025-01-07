# Guide Complet des Collections Java

## Table des matières
1. [Introduction aux Collections](#introduction)
2. [La Généricité en Java](#genericite)
3. [Interface Collection\<E>](#collection)
4. [Les Itérateurs](#iterateurs)
5. [Les Listes](#listes)
6. [Les Ensembles](#ensembles)
7. [Classes Utilitaires](#utilitaires)

<a name="introduction"></a>
## 1. Introduction aux Collections

La bibliothèque des Collections Java fournit un framework complet pour stocker et manipuler des groupes d'objets. Elle offre :

- Des structures de données riches et efficaces
- Des implémentations génériques réutilisables
- Des performances optimisées
- Une organisation hiérarchique claire

### Architecture principale

Il existe deux hiérarchies principales :
1. `Collection<E>` : pour les collections d'éléments
2. `Map<K,V>` : pour les associations clé-valeur

<a name="genericite"></a>
## 2. La Généricité en Java

La généricité permet d'écrire du code qui fonctionne avec différents types tout en conservant la sécurité du typage.

### Exemple de base :

```java
// Déclaration générique
public class Boite<T> {
    private T contenu;
    
    public void mettre(T element) {
        this.contenu = element;
    }
    
    public T obtenir() {
        return contenu;
    }
}

// Utilisation
Boite<String> boiteTexte = new Boite<>();
boiteTexte.mettre("Hello");
String texte = boiteTexte.obtenir(); // Pas besoin de cast

Boite<Integer> boiteNombre = new Boite<>();
boiteNombre.mettre(42);
int nombre = boiteNombre.obtenir(); // Pas besoin de cast
```

<a name="collection"></a>
## 3. Interface Collection\<E>

L'interface `Collection<E>` est la racine de la hiérarchie des collections. Elle définit les opérations de base disponibles sur toutes les collections.

### Méthodes principales :

```java
public interface Collection<E> {
    // Opérations basiques
    boolean add(E element);
    boolean remove(Object o);
    boolean contains(Object o);
    int size();
    boolean isEmpty();
    
    // Opérations en masse
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    void clear();
    
    // Conversion en tableau
    Object[] toArray();
    <T> T[] toArray(T[] a);
}
```

### Exemple pratique :

```java
Collection<String> collection = new ArrayList<>();

// Ajouts
collection.add("Java");
collection.add("Python");
collection.add("JavaScript");

// Vérification
System.out.println("Taille : " + collection.size()); // 3
System.out.println("Contient Java ? " + collection.contains("Java")); // true

// Parcours
for (String langage : collection) {
    System.out.println(langage);
}

// Conversion en tableau
String[] tableauLangages = collection.toArray(new String[0]);
```

<a name="iterateurs"></a>
## 4. Les Itérateurs

Les itérateurs permettent de parcourir et modifier une collection de manière sécurisée.

### Types d'itérateurs :
1. `Iterator<E>` : parcours simple
2. `ListIterator<E>` : parcours bidirectionnel pour les listes

### Exemple d'utilisation d'Iterator :

```java
Collection<String> fruits = new ArrayList<>();
fruits.add("Pomme");
fruits.add("Banane");
fruits.add("Orange");

// Utilisation basique
Iterator<String> it = fruits.iterator();
while (it.hasNext()) {
    String fruit = it.next();
    System.out.println(fruit);
}

// Suppression pendant l'itération
Iterator<String> it2 = fruits.iterator();
while (it2.hasNext()) {
    String fruit = it2.next();
    if (fruit.equals("Banane")) {
        it2.remove(); // Suppression sécurisée
    }
}
```

### Exemple de ListIterator :

```java
List<String> list = new ArrayList<>();
list.add("Un");
list.add("Deux");
list.add("Trois");

ListIterator<String> lit = list.listIterator();

// Parcours avant
while (lit.hasNext()) {
    System.out.println("Index: " + lit.nextIndex() + ", Valeur: " + lit.next());
}

// Parcours arrière
while (lit.hasPrevious()) {
    System.out.println("Index: " + lit.previousIndex() + ", Valeur: " + lit.previous());
}
```

<a name="listes"></a>
## 5. Les Listes

Les listes sont des collections ordonnées qui permettent les doublons.

### Principales implémentations :
1. `ArrayList<E>` : tableau dynamique
2. `LinkedList<E>` : liste doublement chaînée
3. `Vector<E>` : version thread-safe (obsolète)

### Exemple complet ArrayList :

```java
public class GestionEmployes {
    public static void main(String[] args) {
        // Création
        List<String> employes = new ArrayList<>();
        
        // Ajouts
        employes.add("Alice");
        employes.add("Bob");
        employes.add("Charlie");
        employes.add(1, "David"); // Insertion à l'index 1
        
        // Accès
        String premier = employes.get(0); // Alice
        int position = employes.indexOf("Bob"); // 2
        
        // Modification
        employes.set(0, "Alice Smith");
        
        // Sous-liste
        List<String> sousListe = employes.subList(1, 3);
        
        // Tri
        Collections.sort(employes);
        
        // Parcours avec index
        for (int i = 0; i < employes.size(); i++) {
            System.out.println("Employé " + i + ": " + employes.get(i));
        }
    }
}
```

<a name="ensembles"></a>
## 6. Les Ensembles (Set)

Les ensembles sont des collections qui n'acceptent pas les doublons.

### Types principaux :
1. `HashSet<E>` : ensemble basé sur une table de hachage
2. `TreeSet<E>` : ensemble trié
3. `LinkedHashSet<E>` : ensemble qui maintient l'ordre d'insertion

### Exemple complet :

```java
public class GestionEtudiants {
    public static void main(String[] args) {
        // HashSet : ordre aléatoire
        Set<String> etudiants = new HashSet<>();
        etudiants.add("Marc");
        etudiants.add("Sophie");
        etudiants.add("Marc"); // Ignoré car doublon
        System.out.println("HashSet: " + etudiants);
        
        // TreeSet : trié naturellement
        Set<String> etudiantsOrdonnes = new TreeSet<>();
        etudiantsOrdonnes.addAll(etudiants);
        System.out.println("TreeSet: " + etudiantsOrdonnes);
        
        // LinkedHashSet : garde l'ordre d'insertion
        Set<String> etudiantsOrdonnesInsertion = new LinkedHashSet<>();
        etudiantsOrdonnesInsertion.addAll(etudiants);
        System.out.println("LinkedHashSet: " + etudiantsOrdonnesInsertion);
    }
}
```

<a name="utilitaires"></a>
## 7. Classes Utilitaires

### Arrays

La classe `Arrays` fournit des méthodes utilitaires pour les tableaux.

```java
// Exemple d'utilisation de Arrays
int[] nombres = {5, 2, 8, 1, 9};
Arrays.sort(nombres); // Tri
int index = Arrays.binarySearch(nombres, 8); // Recherche binaire
int[] copie = Arrays.copyOf(nombres, nombres.length); // Copie
String representation = Arrays.toString(nombres); // Conversion en String
```

### Collections

La classe `Collections` fournit des méthodes utilitaires pour les collections.

```java
List<String> liste = new ArrayList<>();
liste.add("C");
liste.add("A");
liste.add("B");

// Méthodes utilitaires
Collections.sort(liste); // Tri
Collections.reverse(liste); // Inversion
Collections.shuffle(liste); // Mélange
String min = Collections.min(liste); // Minimum
String max = Collections.max(liste); // Maximum

// Créer des collections immuables
List<String> listeImmuable = Collections.unmodifiableList(liste);
Set<String> ensembleVide = Collections.emptySet();
List<String> listeUnique = Collections.singletonList("Unique");
```

## Bonnes Pratiques

1. **Choix de l'implémentation** :
   - `ArrayList` pour accès aléatoire fréquent
   - `LinkedList` pour insertions/suppressions fréquentes
   - `HashSet` pour recherche rapide sans doublons
   - `TreeSet` pour données triées

2. **Performance** :
   - Initialiser les collections avec une capacité initiale si la taille est connue
   - Utiliser les bonnes méthodes de parcours selon le cas d'usage
   - Éviter les modifications pendant le parcours avec for-each

3. **Sécurité** :
   - Utiliser les collections immuables quand possible
   - Toujours utiliser la généricité
   - Gérer correctement les exceptions ConcurrentModificationException

4. **Maintenance** :
   - Documenter le choix des implémentations
   - Utiliser les interfaces plutôt que les implémentations dans les signatures
   - Suivre les conventions de nommage
