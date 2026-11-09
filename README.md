# Java Generics: Curingas Delimitados e Princípio Get/Put (PECS)

Projeto prático desenvolvido no módulo de Tópicos Especiais / Genéricos do curso de Java Completo do Prof. Nélio Alves (DevSuperior). 

O objetivo desta aplicação é demonstrar o comportamento avançado de **Tipos Genéricos (Generics)** em Java, abordando o problema da **invariância**, o uso de **tipos curinga delimitados** (*bounded wildcards*), **polimorfismo com interfaces** e a aplicação do princípio **Get/Put** formulado como **PECS** (*Producer Extends, Consumer Super*).

---

## 📁 Estrutura do Projeto

O código-fonte organiza-se em dois pacotes fundamentais:

```text
src/
├── application/
│   └── Program.java       # Ponto de entrada, métodos de cópia e impressão universal
└── entities/
    ├── Shape.java         # Interface contratual de formas geométricas
    ├── Circle.java        # Implementação de círculo com cálculo de área
    └── Rectangle.java     # Implementação de retângulo com cálculo de área
```

---

## 🧩 Componentes do Domínio (`entities`)

A hierarquia de formas geométricas ilustra o uso de tipos delimitados em modelos polimórficos:

### 1. `Shape.java`
Interface base que define o contrato funcional para formas geométricas:
```java
package entities;

public interface Shape {
    double area();
}
```

### 2. `Circle.java`
Classe que implementa `Shape`, encapsulando o raio e calculando a área baseada em $\pi \times r^2$:
```java
package entities;

public class Circle implements Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    public double getRadius() {
        return radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

### 3. `Rectangle.java`
Classe que implementa `Shape`, encapsulando largura e altura:
```java
package entities;

public class Rectangle implements Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    public double getWidth() {
        return width;
    }

    public double getHeight() {
        return height;
    }

    @Override
    public double area() {
        return width * height;
    }
}
```

---

## ⚙️ A Aplicação Principal (`Program.java`)

Demonstra a transferência e manipulação de listas numéricas aplicando os curingas delimitados:

```java
package application;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class Program {

    public static void main(String[] args) {
        List<Integer> myInts = Arrays.asList(1, 2, 3, 4);
        List<Double> myDoubles = Arrays.asList(3.14, 6.28);
        List<Object> myObjs = new ArrayList<Object>();

        copy(myInts, myObjs);
        printList(myObjs);

        copy(myDoubles, myObjs);
        printList(myObjs);
    }

    public static void copy(List<? extends Number> source, List<? super Number> destiny) {
        for(Number number : source) {
            destiny.add(number);
        }
    }

    public static void printList(List<?> list) {
        for (Object obj : list) {
            System.out.print(obj + " ");
        }
        System.out.println();
    }
}
```

---

## 🧠 As Regras Fundamentais dos Generics

### 1. Invariância dos Tipos Genéricos
Diferente dos arrays primitivos, os tipos genéricos em Java são **invariantes**:
* Embora `Integer` herde de `Number`, `List<Integer>` **não** herda de `List<Number>`.
* Se isso fosse permitido, poderíamos atribuir uma `List<Integer>` a uma referência `List<Number>` e tentar inserir um `Double`, quebrando a segurança de tipos em tempo de execução (*type safety*).

### 2. Covariância: `? extends T` (Regra do GET)
* **Expressão:** `List<? extends Number> source`
* **Significado:** A lista pode conter elementos de `Number` ou de qualquer subtipo dele (`Integer`, `Double`, etc.).
* **Regra GET:** **Permitido.** Qualquer elemento extraído é garantidamente, no mínimo, um `Number`.
* **Regra PUT:** **Proibido.** Não é permitido adicionar novos itens (com exceção de `null`), pois o compilador não sabe o tipo concreto real da lista referenciada.
* **Papel:** Produtor (*Producer*).

### 3. Contravariância: `? super T` (Regra do PUT)
* **Expressão:** `List<? super Number> destiny`
* **Significado:** A lista pode ser de `Number` ou de qualquer supertipo dele (ex.: `Object`).
* **Regra PUT:** **Permitido.** Qualquer instância de `Number` (ou de suas subclasses) pode ser inserida com segurança.
* **Regra GET:** **Restrito.** Elementos lidos só podem ser capturados como a referência mais genérica: `Object`.
* **Papel:** Consumidor (*Consumer*).

### 4. Princípio PECS (Producer Extends, Consumer Super)
Regra prática de design descrita por Joshua Bloch (*Effective Java*):
* Se a coleção apenas **fornece** dados para o método $\rightarrow$ use `? extends T`.
* Se a coleção apenas **recebe/armazena** dados do método $\rightarrow$ use `? super T`.
* Se o método precisar **ler e escrever** na mesma coleção $\rightarrow$ não utilize curingas delimitados; use tipos exatos como `List<T>`.

### 5. Curinga Não Delimitado: `List<?>`
* Representa um tipo de lista totalmente desconhecido.
* Permite apenas leitura dos dados como `Object`.
* Ideal para métodos puramente utilitários e desacoplados, como o `printList(List<?> list)`.

### 6. Relação com as Entidades (`Shape`)
O princípio de covariância permite reaproveitamento polimórfico de listas de entidades. Por exemplo, para calcular a área total de um grupo de formas:
```java
public static double totalArea(List<? extends Shape> list) {
    double sum = 0.0;
    for (Shape s : list) {
        sum += s.area(); // Permite get seguro devido a '? extends Shape'
    }
    return sum;
}
```
Com essa assinatura, podemos passar tanto `List<Circle>` quanto `List<Rectangle>` diretamente para o método sem erros de compilação.

---

## 📊 Tabela de Decisão Rápida (Get / Put)

| Notação | Conceito | GET (`elemento = list.get()`) | PUT (`list.add(item)`) | Função no Método |
| :--- | :--- | :---: | :---: | :--- |
| `List<? extends T>` | **Covariância** | ✅ Seguro (como `T`) | ❌ Proibido (gera erro de compilação) | Produtor (*Producer*) |
| `List<? super T>` | **Contravariância** | ⚠️ Apenas como `Object` | ✅ Seguro (adiciona `T` ou subclasses) | Consumidor (*Consumer*) |
| `List<?>` | **Não Delimitado** | ⚠️ Apenas como `Object` | ❌ Proibido (apenas `null`) | Leitor neutro |

---

## 🖥️ Saída Esperada no Console

Ao executar o método `main` da classe `Program`:

```text
1 2 3 4 
1 2 3 4 3.14 6.28 
```

---

## 🚀 Como Compilar e Executar

### Pré-requisitos
* Java Development Kit (JDK) 11 ou superior instalado.

### Via Linha de Comando (Terminal)

1. No diretório raiz do projeto (onde ficam as pastas `application` e `entities`):

```bash
# Compilar todas as classes para a pasta bin
javac -d bin entities/*.java application/*.java

# Executar a aplicação
java -cp bin application.Program
```

### Via IDE (Eclipse, IntelliJ IDEA, VS Code)
1. Importe o diretório como um projeto Java comum.
2. Abra o arquivo `src/application/Program.java`.
3. Execute através do comando **Run** / **Run As Java Application**.