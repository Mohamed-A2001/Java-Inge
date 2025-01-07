# Guide complet des entrées/sorties en Java

## Table des matières
1. [Introduction aux entrées/sorties en Java](#1-introduction)
2. [Lecture de fichiers texte](#2-lecture-de-fichiers-texte)
3. [Manipulation des flux binaires](#3-manipulation-des-flux-binaires)
4. [Classes orientées fichiers](#4-classes-orientées-fichiers)
5. [Flux avancés et buffers](#5-flux-avancés-et-buffers)
6. [Annexe : Outils supplémentaires](#6-annexe--outils-supplémentaires)

## 1. Introduction

Les entrées/sorties (I/O) en Java permettent de lire et écrire des données depuis différentes sources :
- Fichiers
- Flux de données
- Connexions réseau
- etc.

Le système d'I/O de Java est organisé autour de plusieurs concepts clés :

- **Flux (Streams)** : Représentent une séquence de données
- **Lecteurs/Écrivains (Readers/Writers)** : Pour manipuler du texte
- **Flux binaires (Binary Streams)** : Pour manipuler des données brutes

## 2. Lecture de fichiers texte

### 2.1 StreamTokenizer

Le `StreamTokenizer` est un outil puissant pour lire des fichiers texte structurés. Il découpe le texte en "tokens" (unités lexicales).

Exemple complet d'utilisation :

```java
import java.io.*;

public class TokenizerExample {
    public static void main(String[] args) throws IOException {
        // Création du tokenizer
        StreamTokenizer tokenizer = new StreamTokenizer(
            new BufferedReader(new FileReader("input.txt"))
        );
        
        // Configuration
        tokenizer.resetSyntax(); // Reset de la configuration par défaut
        tokenizer.wordChars('a', 'z');
        tokenizer.wordChars('A', 'Z');
        tokenizer.whitespaceChars(0, ' ');
        tokenizer.parseNumbers();
        
        // Lecture
        while (tokenizer.nextToken() != StreamTokenizer.TT_EOF) {
            switch(tokenizer.ttype) {
                case StreamTokenizer.TT_NUMBER:
                    System.out.println("Nombre: " + tokenizer.nval);
                    break;
                case StreamTokenizer.TT_WORD:
                    System.out.println("Mot: " + tokenizer.sval);
                    break;
                default:
                    System.out.println("Autre: " + (char)tokenizer.ttype);
            }
        }
    }
}
```

### 2.2 Structures de lecture type "look-ahead"

Cette technique permet de lire un élément à l'avance pour prendre des décisions sur le traitement :

```java
public class LookAheadExample {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
        String line;
        String nextLine = reader.readLine();
        
        while (nextLine != null) {
            line = nextLine;
            nextLine = reader.readLine();
            
            // On peut prendre des décisions basées sur la ligne suivante
            if (nextLine != null && nextLine.startsWith("ERROR:")) {
                System.out.println("Warning: Error follows: " + line);
            }
        }
        reader.close();
    }
}
```

## 3. Manipulation des flux binaires

### 3.1 InputStream et OutputStream

Ces classes sont la base des opérations binaires en Java.

Exemple de copie de fichier binaire :

```java
public class BinaryFileCopy {
    public static void copy(String source, String dest) throws IOException {
        try (FileInputStream in = new FileInputStream(source);
             FileOutputStream out = new FileOutputStream(dest)) {
            
            byte[] buffer = new byte[1024];
            int length;
            
            while ((length = in.read(buffer)) > 0) {
                out.write(buffer, 0, length);
            }
        }
    }
}
```

### 3.2 DataInputStream et DataOutputStream

Pour lire/écrire des types de données primitifs :

```java
public class DataStreamExample {
    public static void main(String[] args) throws IOException {
        // Écriture
        try (DataOutputStream out = new DataOutputStream(
                new FileOutputStream("data.bin"))) {
            out.writeInt(42);
            out.writeDouble(3.14);
            out.writeUTF("Hello");
        }
        
        // Lecture
        try (DataInputStream in = new DataInputStream(
                new FileInputStream("data.bin"))) {
            int i = in.readInt();
            double d = in.readDouble();
            String s = in.readUTF();
            System.out.printf("Lu: %d, %f, %s%n", i, d, s);
        }
    }
}
```

## 4. Classes orientées fichiers

### 4.1 La classe File

Représente un fichier ou un répertoire sur le système de fichiers.

```java
public class FileExample {
    public static void main(String[] args) {
        File directory = new File("./documents");
        
        // Création d'un répertoire
        if (!directory.exists()) {
            directory.mkdirs();
        }
        
        // Liste des fichiers
        File[] files = directory.listFiles();
        if (files != null) {
            for (File file : files) {
                System.out.printf("Nom: %s, Taille: %d bytes%n", 
                    file.getName(), file.length());
            }
        }
        
        // Filtre personnalisé
        FilenameFilter pdfFilter = (dir, name) -> name.endsWith(".pdf");
        File[] pdfFiles = directory.listFiles(pdfFilter);
    }
}
```

### 4.2 Path et Files (Java NIO)

L'API moderne pour la manipulation de fichiers :

```java
import java.nio.file.*;

public class PathExample {
    public static void main(String[] args) throws IOException {
        Path path = Paths.get("test.txt");
        
        // Écriture de fichier
        Files.write(path, "Hello World".getBytes());
        
        // Lecture de fichier
        List<String> lines = Files.readAllLines(path);
        
        // Copie de fichier
        Path source = Paths.get("source.txt");
        Path target = Paths.get("target.txt");
        Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
        
        // Parcours d'un répertoire
        Path dir = Paths.get(".");
        try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir)) {
            for (Path entry : stream) {
                System.out.println(entry.getFileName());
            }
        }
    }
}
```

## 5. Flux avancés et buffers

### 5.1 BufferedReader et BufferedWriter

Ces classes ajoutent une mise en buffer pour améliorer les performances :

```java
public class BufferedIOExample {
    public static void main(String[] args) throws IOException {
        // Écriture
        try (BufferedWriter writer = new BufferedWriter(
                new FileWriter("output.txt"))) {
            writer.write("Première ligne");
            writer.newLine();
            writer.write("Deuxième ligne");
        }
        
        // Lecture
        try (BufferedReader reader = new BufferedReader(
                new FileReader("output.txt"))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        }
    }
}
```

### 5.2 ByteBuffer (NIO)

Pour des opérations avancées sur les données binaires :

```java
import java.nio.*;

public class ByteBufferExample {
    public static void main(String[] args) {
        // Création d'un buffer
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        
        // Écriture dans le buffer
        buffer.putInt(42);
        buffer.putDouble(3.14);
        
        // Préparation pour la lecture
        buffer.flip();
        
        // Lecture depuis le buffer
        int i = buffer.getInt();
        double d = buffer.getDouble();
        
        System.out.printf("Lu: %d, %f%n", i, d);
    }
}
```

## 6. Annexe : Outils supplémentaires

### 6.1 Scanner

Une alternative plus simple pour la lecture formatée :

```java
import java.util.Scanner;

public class ScannerExample {
    public static void main(String[] args) {
        String input = "1 2 3 4 5";
        Scanner scanner = new Scanner(input);
        
        // Lecture de nombres
        while (scanner.hasNextInt()) {
            int n = scanner.nextInt();
            System.out.println("Nombre lu: " + n);
        }
        
        // Exemple avec un fichier
        try (Scanner fileScanner = new Scanner(new File("data.txt"))) {
            // Configuration personnalisée
            fileScanner.useDelimiter(",");
            
            while (fileScanner.hasNext()) {
                String token = fileScanner.next();
                System.out.println("Token: " + token.trim());
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

### 6.2 Sérialisation

Pour sauvegarder et charger des objets Java :

```java
import java.io.*;

// Classe à sérialiser
class Person implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}

public class SerializationExample {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Person person = new Person("Alice", 30);
        
        // Sérialisation
        try (ObjectOutputStream out = new ObjectOutputStream(
                new FileOutputStream("person.dat"))) {
            out.writeObject(person);
        }
        
        // Désérialisation
        try (ObjectInputStream in = new ObjectInputStream(
                new FileInputStream("person.dat"))) {
            Person loaded = (Person) in.readObject();
            System.out.println("Personne chargée: " + loaded);
        }
    }
}
```

### 6.3 Gestion des caractères spéciaux et encodages

```java
public class EncodingExample {
    public static void main(String[] args) throws IOException {
        // Écriture avec encodage spécifique
        try (Writer writer = new OutputStreamWriter(
                new FileOutputStream("utf8.txt"), "UTF-8")) {
            writer.write("é à ç ê");
        }
        
        // Lecture avec le même encodage
        try (Reader reader = new InputStreamReader(
                new FileInputStream("utf8.txt"), "UTF-8")) {
            char[] buffer = new char[1024];
            int read = reader.read(buffer);
            System.out.println(new String(buffer, 0, read));
        }
    }
}
```

Ce guide couvre les aspects essentiels des entrées/sorties en Java avec des exemples pratiques. N'hésitez pas à adapter ces exemples selon vos besoins spécifiques.
