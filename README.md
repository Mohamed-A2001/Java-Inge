# Entrées/Sorties en Java

## Table des matières
- [1. Introduction](#1-introduction)
- [2. Les flux (streams)](#2-les-flux-streams)
- [3. La hiérarchie des classes d'entrées/sorties](#3-la-hiérarchie-des-classes-dentréessorties)
- [4. Lecture de texte](#4-lecture-de-texte)
- [5. Écriture de texte](#5-écriture-de-texte)
- [6. Gestion des ressources](#6-gestion-des-ressources)
- [7. Cas pratiques](#7-cas-pratiques)

## 1. Introduction

Les entrées/sorties (I/O) sont les opérations permettant à un programme d'échanger des données avec l'extérieur (fichiers, réseau, périphériques...).

### Définitions

- **Entrée (Input)** : opération permettant à une machine de récupérer (lire) une donnée depuis un périphérique (webcam, réseau, fichier sur disque...)
- **Sortie (Output)** : opération permettant à une machine d'envoyer (écrire) une donnée vers un périphérique (écran, LED, fichier sur disque, réseau...)
- **I/O** : abréviation pour Input/Output (entrées/sorties)

## 2. Les flux (streams)

Un flux est une séquence de données (finie ou infinie) dotée d'un curseur. Les opérations de lecture et d'écriture se font à la position du curseur.

### Caractéristiques des flux

- Suite de données avec un curseur de position
- Deux types principaux : lecture et écriture 
- Les opérations avancent automatiquement le curseur
- Position initiale avant la première donnée

### Exemple de manipulation d'un flux texte

```java
// Lecture caractère par caractère
FileReader reader = new FileReader("input.txt");
int c;
while ((c = reader.read()) != -1) {
    System.out.print((char)c);
}
reader.close();
```

## 3. La hiérarchie des classes d'entrées/sorties

### Classes de base pour la lecture

- `Reader` : classe abstraite de base pour la lecture de texte
- `FileReader` : lecture depuis un fichier
- `StringReader` : lecture depuis une chaîne de caractères
- `BufferedReader` : ajoute la mise en tampon et la lecture ligne par ligne

### Classes de base pour l'écriture 

- `Writer` : classe abstraite de base pour l'écriture de texte
- `FileWriter` : écriture dans un fichier
- `StringWriter` : écriture dans une chaîne de caractères
- `BufferedWriter` : ajoute la mise en tampon

### Exemple complet utilisant différentes classes

```java
public class IODemo {
    public static void main(String[] args) throws IOException {
        // Lecture depuis un fichier
        try (BufferedReader fileReader = new BufferedReader(new FileReader("input.txt"))) {
            String line;
            while ((line = fileReader.readLine()) != null) {
                System.out.println(line);
            }
        }
        
        // Lecture depuis une chaîne
        String data = "Test de lecture\nSur plusieurs lignes";
        try (BufferedReader stringReader = new BufferedReader(new StringReader(data))) {
            String line;
            while ((line = stringReader.readLine()) != null) {
                System.out.println(line);
            }
        }
    }
}
```

## 4. Lecture de texte

### FileReader

Utilisation basique :

```java
FileReader reader = null;
try {
    reader = new FileReader("test.txt");
    int c;
    while ((c = reader.read()) != -1) {
        System.out.print((char)c);
    }
} finally {
    if (reader != null) {
        reader.close();
    }
}
```

### BufferedReader

Lecture ligne par ligne avec mise en tampon :

```java
try (BufferedReader reader = new BufferedReader(new FileReader("test.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}
```

## 5. Écriture de texte

### FileWriter

Écriture simple dans un fichier :

```java
try (FileWriter writer = new FileWriter("output.txt")) {
    writer.write("Première ligne\n");
    writer.write("Deuxième ligne\n");
}
```

### BufferedWriter

Écriture avec tampon :

```java
try (BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
    writer.write("Ligne 1");
    writer.newLine();
    writer.write("Ligne 2");
    writer.newLine();
    writer.flush(); // Force l'écriture du tampon
}
```

## 6. Gestion des ressources

### Utilisation de try-with-resources

Depuis Java 7, la meilleure approche pour gérer les ressources :

```java
try (FileReader reader = new FileReader("input.txt");
     FileWriter writer = new FileWriter("output.txt")) {
    
    int c;
    while ((c = reader.read()) != -1) {
        writer.write(c);
    }
}
```

### Ancien style (avant Java 7)

```java
FileReader reader = null;
FileWriter writer = null;
try {
    reader = new FileReader("input.txt");
    writer = new FileWriter("output.txt");
    
    int c;
    while ((c = reader.read()) != -1) {
        writer.write(c);
    }
} finally {
    if (writer != null) {
        try {
            writer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 7. Cas pratiques

### Exemple 1: Copie de fichier

```java
public class FileCopy {
    public static void copy(String source, String destination) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(source));
             BufferedWriter writer = new BufferedWriter(new FileWriter(destination))) {
            
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }
        }
    }
}
```

### Exemple 2: Comptage de mots

```java
public class WordCounter {
    public static Map<String, Integer> countWords(String filename) throws IOException {
        Map<String, Integer> wordCount = new HashMap<>();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] words = line.split("\\s+");
                for (String word : words) {
                    word = word.toLowerCase().trim();
                    if (!word.isEmpty()) {
                        wordCount.merge(word, 1, Integer::sum);
                    }
                }
            }
        }
        
        return wordCount;
    }
}
```

### Exemple 3: Lecture formatée d'un fichier CSV

```java
public class CSVReader {
    public static List<Person> readPeople(String filename) throws IOException {
        List<Person> people = new ArrayList<>();
        
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            // Skip header
            reader.readLine();
            
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split(",");
                if (parts.length >= 3) {
                    String name = parts[0].trim();
                    int age = Integer.parseInt(parts[1].trim());
                    String city = parts[2].trim();
                    people.add(new Person(name, age, city));
                }
            }
        }
        
        return people;
    }
    
    public static class Person {
        private final String name;
        private final int age;
        private final String city;
        
        public Person(String name, int age, String city) {
            this.name = name;
            this.age = age;
            this.city = city;
        }
        
        // Getters omitted for brevity
    }
}
```

### Exemple 4: Lecture d'une heure au format "HH:mm"

```java
public class TimeParser {
    public static LocalTime parseTime(String input) {
        try (StringReader reader = new StringReader(input)) {
            StringBuilder hours = new StringBuilder();
            StringBuilder minutes = new StringBuilder();
            int c;
            
            // Lecture des heures
            while ((c = reader.read()) != -1 && c != ':') {
                if (!Character.isDigit(c)) {
                    throw new IllegalArgumentException("Format invalide");
                }
                hours.append((char)c);
            }
            
            // Lecture des minutes
            while ((c = reader.read()) != -1) {
                if (!Character.isDigit(c)) {
                    throw new IllegalArgumentException("Format invalide");
                }
                minutes.append((char)c);
            }
            
            int h = Integer.parseInt(hours.toString());
            int m = Integer.parseInt(minutes.toString());
            
            if (h < 0 || h > 23 || m < 0 || m > 59) {
                throw new IllegalArgumentException("Valeurs invalides");
            }
            
            return LocalTime.of(h, m);
        } catch (IOException e) {
            throw new IllegalArgumentException("Erreur de lecture", e);
        }
    }
}
```

### Points importants à retenir

1. Toujours fermer les ressources (utiliser try-with-resources)
2. Gérer correctement les exceptions
3. Utiliser les classes bufferisées pour de meilleures performances
4. Penser à la gestion des encodages pour les fichiers texte
5. Valider les données lues
6. Utiliser les classes appropriées selon le besoin (Reader vs InputStream)

