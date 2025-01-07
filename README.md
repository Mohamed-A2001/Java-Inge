# Guide Complet des Nouveautés Java 8+

## Table des matières
- [Méthodes par défaut dans les interfaces](#méthodes-par-défaut-dans-les-interfaces)
- [Lambda expressions](#lambda-expressions)
- [Interfaces fonctionnelles](#interfaces-fonctionnelles)
- [Streams](#streams)
- [Optional](#optional)
- [Nouveautés des versions suivantes](#nouveautés-des-versions-suivantes)

## Méthodes par défaut dans les interfaces

Java 8 introduit le concept de méthodes par défaut dans les interfaces, permettant d'ajouter des implémentations concrètes aux interfaces sans briser la compatibilité avec le code existant.

### Caractéristiques principales
- Une interface peut contenir des implémentations de méthodes (avec le mot-clé `default`)
- Pas de variables d'instance possibles
- Les classes qui implémentent l'interface n'ont pas à réimplémenter ces méthodes
- En cas de conflit (héritage multiple), la classe doit redéfinir la méthode

### Exemple complet

```java
public interface Sortable<T> {
    // Méthode abstraite traditionnelle
    int compare(T a, T b);
    
    // Méthode par défaut utilisant la méthode abstraite
    default void sort(List<T> list) {
        list.sort((a, b) -> compare(a, b));
    }
    
    // Autre méthode par défaut
    default boolean isSorted(List<T> list) {
        if (list.size() <= 1) return true;
        
        for (int i = 1; i < list.size(); i++) {
            if (compare(list.get(i-1), list.get(i)) > 0) {
                return false;
            }
        }
        return true;
    }
}

// Implémentation
public class NumberSorter implements Sortable<Integer> {
    @Override
    public int compare(Integer a, Integer b) {
        return a.compareTo(b);
    }
    // Pas besoin d'implémenter sort() et isSorted()
}
```

## Lambda expressions

Les lambda expressions permettent d'écrire des fonctions anonymes de manière concise.

### Syntaxe de base
```java
// Sans paramètres
Runnable r = () -> System.out.println("Hello");

// Un seul paramètre
Consumer<String> printer = s -> System.out.println(s);

// Plusieurs paramètres
BinaryOperator<Integer> sum = (a, b) -> a + b;

// Corps avec plusieurs instructions
Comparator<String> comp = (s1, s2) -> {
    if (s1 == null) return -1;
    if (s2 == null) return 1;
    return s1.compareTo(s2);
};
```

### Exemple pratique : Filtrage de liste

```java
public class PersonFilter {
    public static void main(String[] args) {
        List<Person> persons = Arrays.asList(
            new Person("Alice", 25),
            new Person("Bob", 30),
            new Person("Charlie", 35)
        );

        // Java 7
        List<Person> adults = new ArrayList<>();
        for (Person p : persons) {
            if (p.getAge() >= 18) {
                adults.add(p);
            }
        }

        // Java 8
        List<Person> adults8 = persons.stream()
            .filter(p -> p.getAge() >= 18)
            .collect(Collectors.toList());
    }
}
```

## Interfaces fonctionnelles

Une interface fonctionnelle est une interface qui ne possède qu'une seule méthode abstraite. Java 8 fournit plusieurs interfaces fonctionnelles standard.

### Interfaces fonctionnelles communes

```java
// Function<T,R> : transforme T en R
Function<String, Integer> length = String::length;

// Predicate<T> : test qui retourne un booléen
Predicate<String> isEmpty = String::isEmpty;

// Consumer<T> : accepte un argument mais ne retourne rien
Consumer<String> printer = System.out::println;

// Supplier<T> : ne prend pas d'argument mais produit une valeur
Supplier<LocalDate> today = LocalDate::now;

// UnaryOperator<T> : function qui prend et retourne le même type
UnaryOperator<String> upper = String::toUpperCase;

// BinaryOperator<T> : combine deux valeurs du même type
BinaryOperator<String> concat = String::concat;
```

### Exemple complet d'utilisation

```java
public class FunctionalExample {
    public static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
        List<T> result = new ArrayList<>();
        for (T item : list) {
            if (predicate.test(item)) {
                result.add(item);
            }
        }
        return result;
    }

    public static <T, R> List<R> map(List<T> list, Function<T, R> mapper) {
        List<R> result = new ArrayList<>();
        for (T item : list) {
            result.add(mapper.apply(item));
        }
        return result;
    }

    public static void main(String[] args) {
        List<String> words = Arrays.asList("hello", "", "world", null, "java");

        // Filtrer les chaînes non nulles et non vides
        Predicate<String> notEmpty = s -> s != null && !s.isEmpty();
        List<String> nonEmpty = filter(words, notEmpty);

        // Transformer en majuscules
        Function<String, String> toUpper = String::toUpperCase;
        List<String> upperCase = map(nonEmpty, toUpper);

        // Afficher chaque mot
        Consumer<String> print = System.out::println;
        upperCase.forEach(print);
    }
}
```

## Streams

Les streams permettent de traiter des séquences d'éléments de manière déclarative et potentiellement parallèle.

### Opérations courantes

```java
List<String> words = Arrays.asList("hello", "world", "java", "stream");

// filter
List<String> longWords = words.stream()
    .filter(w -> w.length() > 4)
    .collect(Collectors.toList());

// map
List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());

// reduce
int totalLength = words.stream()
    .mapToInt(String::length)
    .sum();

// collect avec grouping
Map<Integer, List<String>> byLength = words.stream()
    .collect(Collectors.groupingBy(String::length));

// parallel stream
long count = words.parallelStream()
    .filter(w -> w.length() > 4)
    .count();
```

### Exemple avancé : Traitement de données

```java
public class OrderProcessor {
    public static class Order {
        private String customer;
        private List<OrderLine> lines;
        private LocalDate date;

        // constructeurs et getters
    }

    public static class OrderLine {
        private String product;
        private int quantity;
        private BigDecimal unitPrice;

        // constructeurs et getters
    }

    public static void processOrders(List<Order> orders) {
        // Calcul du total par client
        Map<String, BigDecimal> totalByCustomer = orders.stream()
            .collect(Collectors.groupingBy(
                Order::getCustomer,
                Collectors.flatMapping(
                    order -> order.getLines().stream(),
                    Collectors.reducing(
                        BigDecimal.ZERO,
                        line -> line.getUnitPrice()
                            .multiply(new BigDecimal(line.getQuantity())),
                        BigDecimal::add
                    )
                )
            ));

        // Produits les plus vendus
        Map<String, Long> productQuantities = orders.stream()
            .flatMap(order -> order.getLines().stream())
            .collect(Collectors.groupingBy(
                OrderLine::getProduct,
                Collectors.summingLong(OrderLine::getQuantity)
            ));

        // Top 5 des produits
        List<Map.Entry<String, Long>> topProducts = productQuantities.entrySet()
            .stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
            .limit(5)
            .collect(Collectors.toList());
    }
}
```

## Optional

Optional est un conteneur qui peut contenir ou non une valeur non nulle.

### Utilisation de base

```java
public class OptionalExample {
    public static Optional<String> findUser(String id) {
        // Simulation d'accès à une base de données
        if ("admin".equals(id)) {
            return Optional.of("Administrator");
        }
        return Optional.empty();
    }

    public static void main(String[] args) {
        // Création
        Optional<String> empty = Optional.empty();
        Optional<String> value = Optional.of("Hello");
        Optional<String> nullable = Optional.ofNullable(null);

        // Vérification et accès
        if (value.isPresent()) {
            System.out.println("Value: " + value.get());
        }

        // orElse et orElseGet
        String result = empty.orElse("Default");
        String lazy = empty.orElseGet(() -> expensiveOperation());

        // map et flatMap
        Optional<Integer> length = value.map(String::length);

        // filter
        Optional<String> filtered = value.filter(s -> s.length() > 5);

        // ifPresent
        value.ifPresent(System.out::println);

        // Chaînage complexe
        findUser("admin")
            .filter(name -> name.length() > 5)
            .map(String::toUpperCase)
            .ifPresent(System.out::println);
    }

    private static String expensiveOperation() {
        // Simulation d'opération coûteuse
        return "Computed Value";
    }
}
```

### Exemple pratique : Service avec Optional

```java
public class UserService {
    private Map<String, User> users = new HashMap<>();

    public Optional<User> findById(String id) {
        return Optional.ofNullable(users.get(id));
    }

    public String getUserStatus(String id) {
        return findById(id)
            .map(User::getStatus)
            .orElse("Unknown");
    }

    public void sendEmailToActiveUser(String id) {
        findById(id)
            .filter(user -> "ACTIVE".equals(user.getStatus()))
            .ifPresent(this::sendEmail);
    }

    public User getOrCreateUser(String id) {
        return findById(id)
            .orElseGet(() -> {
                User newUser = new User(id);
                users.put(id, newUser);
                return newUser;
            });
    }

    private void sendEmail(User user) {
        // Logique d'envoi d'email
    }
}
```

## Nouveautés des versions suivantes

### Java 9
- Modules (Project Jigsaw)
- Collection Factory Methods
```java
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map = Map.of("a", 1, "b", 2);
```

### Java 10
- Type Inference avec var
```java
var list = new ArrayList<String>();
var map = new HashMap<String, String>();
```

### Java 11
- Méthodes String
```java
"hello".isBlank();
"hello\nworld".lines();
"  hello  ".strip();
```

### Java 14
- Switch Expressions
```java
String result = switch (day) {
    case MONDAY, FRIDAY -> "Working day";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Unknown";
};
```

### Java 16
- Records
```java
public record Person(String name, int age) {
    // Constructeur compact avec validation
    public Person {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
    }
}
```

### Java 17
- Sealed Classes
```java
public sealed class Shape 
    permits Circle, Rectangle, Triangle {
    // code commun
}

public final class Circle extends Shape {
    private final double radius;
    // implémentation
}
```

Ce guide couvre les principales nouveautés de Java 8 et des versions suivantes avec des exemples pratiques et détaillés.
