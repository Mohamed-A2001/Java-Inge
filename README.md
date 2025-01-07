# Guide Complet des Exceptions en Java

## Table des matières
- [1. Types d'erreurs à l'exécution](#1-types-derreurs-à-lexécution)
- [2. Mécanismes de gestion d'erreurs](#2-mécanismes-de-gestion-derreurs)
- [3. Structure try-catch-finally](#3-structure-try-catch-finally)
- [4. Hiérarchie des exceptions](#4-hiérarchie-des-exceptions)
- [5. Bonnes pratiques](#5-bonnes-pratiques)
- [6. Exceptions déclarées vs Runtime](#6-exceptions-déclarées-vs-runtime)
- [7. Points clés à retenir](#7-points-clés-à-retenir)

## 1. Types d'erreurs à l'exécution

### 1.1 Erreurs inévitables (externes)
- Fichier introuvable
- Réseau en panne
- Librairie externe buggée
- Problèmes d'E/S
- Erreurs de base de données

### 1.2 Erreurs évitables (internes)
- Division par zéro
- Dépassement d'index de tableau
- Pointeur null
- Cas non prévus dans la logique métier

## 2. Mécanismes de gestion d'erreurs

### 2.1 Approche par codes d'erreur
```java
public int diviser(int a, int b) {
    if (b == 0) {
        return -1; // Code d'erreur
    }
    return a / b;
}

// Utilisation problématique
int resultat = diviser(10, 0);
if (resultat == -1) {
    System.out.println("Erreur: division par zéro");
} else {
    System.out.println("Résultat: " + resultat);
}
```

Problèmes :
- Mélange logique métier et gestion d'erreurs
- Obligation de vérifier les retours
- Confusion possible entre valeur valide et code d'erreur

### 2.2 Approche par exceptions
```java
public int diviser(int a, int b) throws ArithmeticException {
    if (b == 0) {
        throw new ArithmeticException("Division par zéro");
    }
    return a / b;
}

// Utilisation plus claire
try {
    int resultat = diviser(10, 0);
    System.out.println("Résultat: " + resultat);
} catch (ArithmeticException e) {
    System.out.println("Erreur: " + e.getMessage());
}
```

## 3. Structure try-catch-finally

### 3.1 Structure de base
```java
try {
    // Code susceptible de générer une exception
} catch (ExceptionType1 e1) {
    // Traitement de l'exception de type 1
} catch (ExceptionType2 e2) {
    // Traitement de l'exception de type 2
} finally {
    // Code exécuté dans tous les cas
}
```

### 3.2 Exemple complet avec ressources
```java
public class FileProcessor {
    public static String readFile(String path) throws IOException {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader(path));
            StringBuilder content = new StringBuilder();
            String line;
            
            while ((line = reader.readLine()) != null) {
                content.append(line).append("\n");
            }
            return content.toString();
            
        } catch (FileNotFoundException e) {
            System.err.println("Le fichier n'existe pas: " + path);
            throw e;
        } catch (IOException e) {
            System.err.println("Erreur de lecture: " + e.getMessage());
            throw e;
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    System.err.println("Erreur lors de la fermeture: " + e.getMessage());
                }
            }
        }
    }
}
```

### 3.3 Try-with-resources (Java 7+)
```java
public static String readFileModern(String path) throws IOException {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        StringBuilder content = new StringBuilder();
        String line;
        while ((line = reader.readLine()) != null) {
            content.append(line).append("\n");
        }
        return content.toString();
    }
}
```

## 4. Hiérarchie des exceptions

```java
public class CustomBusinessException extends Exception {
    private final String errorCode;
    
    public CustomBusinessException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}

// Utilisation
try {
    // Code métier
    throw new CustomBusinessException("Données invalides", "ERR001");
} catch (CustomBusinessException e) {
    System.err.printf("Erreur %s: %s%n", e.getErrorCode(), e.getMessage());
} catch (Exception e) {
    System.err.println("Erreur générique: " + e.getMessage());
}
```

## 5. Bonnes pratiques

### 5.1 À éviter
```java
// ❌ Mauvais : catch vide
try {
    riskyOperation();
} catch (Exception e) {
    // Ne rien faire
}

// ❌ Mauvais : catch trop générique
try {
    riskyOperation();
} catch (Exception e) {
    e.printStackTrace();
}
```

### 5.2 Bonnes pratiques
```java
// ✅ Bon : gestion spécifique des exceptions
try {
    riskyOperation();
} catch (FileNotFoundException e) {
    log.error("Fichier non trouvé: {}", e.getMessage());
    notifyAdmin("Fichier manquant", e);
} catch (IOException e) {
    log.error("Erreur d'E/S: {}", e.getMessage());
    throw new ServiceException("Erreur lors du traitement", e);
}

// ✅ Bon : utilisation de tests plutôt que d'exceptions quand possible
// Au lieu de:
try {
    if (map.get("key").equals("value")) { }
} catch (NullPointerException e) { }

// Préférer:
if (map.containsKey("key") && "value".equals(map.get("key"))) { }
```

## 6. Exceptions déclarées vs Runtime

### 6.1 Exceptions déclarées
```java
public void methodWithCheckedException() throws IOException {
    Files.readAllLines(Paths.get("file.txt"));
}
```

### 6.2 Runtime Exceptions
```java
public void methodWithRuntimeException() {
    Objects.requireNonNull(someObject, "L'objet ne peut pas être null");
    // Pas besoin de déclarer NullPointerException
}
```

## 7. Points clés à retenir

1. Les exceptions servent à gérer les cas d'erreur, pas le flux normal du programme
2. Toujours fermer les ressources dans un bloc finally ou utiliser try-with-resources
3. Attraper les exceptions les plus spécifiques possible
4. Documenter les exceptions dans la JavaDoc
5. Logger les exceptions de manière appropriée
6. Ne pas oublier que finally s'exécute toujours
7. Éviter d'utiliser les exceptions pour le contrôle de flux
8. Privilégier les tests conditionnels quand c'est possible

## Exemple complet avec logging et documentation

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Service de gestion des utilisateurs.
 */
public class UserService {
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);
    private final UserRepository userRepository;
    private final EmailService emailService;

    /**
     * Créé un nouvel utilisateur dans le système.
     *
     * @param userData Les données de l'utilisateur à créer
     * @return L'utilisateur créé
     * @throws ValidationException Si les données sont invalides
     * @throws DuplicateUserException Si un utilisateur avec cet email existe déjà
     * @throws ServiceException Si une erreur technique survient
     */
    public User createUser(UserData userData) 
            throws ValidationException, DuplicateUserException, ServiceException {
        try {
            // Validation
            if (!isValid(userData)) {
                logger.warn("Tentative de création d'utilisateur avec données invalides: {}", userData);
                throw new ValidationException("Données utilisateur invalides");
            }

            // Vérification doublon
            if (userRepository.existsByEmail(userData.getEmail())) {
                logger.info("Tentative de création d'un utilisateur en doublon: {}", userData.getEmail());
                throw new DuplicateUserException("Email déjà utilisé");
            }

            // Création
            User user = userRepository.save(userData.toUser());
            
            // Notification
            try {
                emailService.sendWelcomeEmail(user);
            } catch (EmailException e) {
                // Log l'erreur mais continue le processus
                logger.error("Erreur d'envoi d'email de bienvenue à {}", user.getEmail(), e);
            }

            logger.info("Nouvel utilisateur créé: {}", user.getId());
            return user;

        } catch (DatabaseException e) {
            logger.error("Erreur lors de la création de l'utilisateur", e);
            throw new ServiceException("Erreur technique lors de la création", e);
        }
    }
}
```

Ce code illustre plusieurs bonnes pratiques :
- Documentation claire des exceptions possibles
- Logging approprié à chaque niveau
- Gestion différenciée selon le type d'erreur
- Encapsulation des exceptions techniques en exceptions métier
- Utilisation de messages d'erreur explicites
