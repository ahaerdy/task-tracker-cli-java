## Progresso - Task Tracker CLI (Java)

### Fase: Passo 2 — Criando o Modelo: `Task.java`

Nesta etapa, construímos o modelo de dados que serve como alicerce do projeto. A classe `Task` define o comportamento e a estrutura de uma tarefa na memória, além de possuir os mecanismos para se auto-converter em texto JSON e exibir-se de forma organizada no terminal.

---

### 1. Estrutura Inicial e a Constante de Formatação

```java
package tasktracker;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Task {

    public static final DateTimeFormatter FORMATTER =
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

```

* **`package tasktracker;`**: Define o escopo organizacional do arquivo.
* **`import`**: Importa a biblioteca de tempo do Java.
* **`public static final DateTimeFormatter FORMATTER`**: Cria uma constante universal acessível por todo o projeto. Ela define a "máscara" visual (`yyyy-MM-dd HH:mm:ss`) para que as datas fiquem fáceis de ler por humanos, impedindo o formato bruto do Java (ex: `2026-05-27T19:10:19`).

```

```


---

### 2. Atributos Privados (Campos de Dados)

```java
    private int id;
    private String description;
    private String status;      
    private String createdAt;   
    private String updatedAt;   

```

* **`private`**: Aplica o conceito de **Encapsulamento**. Protege os dados contra alterações acidentais de fora da classe.
* **`String status`**: Armazenará os estados literais `"todo"`, `"in-progress"` ou `"done"`.
* **`String createdAt` e `updatedAt**`: Guardam o texto da data e hora já devidamente formatado pela constante, facilitando a gravação direta em arquivos.

Anatomia Exata da Declaração:

```plaintext
ssss
```

---

### 3. Sobrecarga de Construtores (Polimorfismo)

O Java permite definir mais de um construtor para a mesma classe, desde que eles recebam parâmetros diferentes. Isso se chama **Sobrecarga**.

#### Construtor 1: Para NOVAS tarefas

```java
    public Task(int id, String description) {
        this.id = id;
        this.description = description;
        this.status = "todo";                                   
        String agora = LocalDateTime.now().format(FORMATTER);  
        this.createdAt = agora;
        this.updatedAt = agora;
    }

```

* **Utilidade**: Usado quando o usuário digita um comando para criar uma tarefa inédita no sistema.
* **Lógica**: O status é forçado para `"todo"`. O sistema captura o instante exato com `LocalDateTime.now()`, aplica a máscara e salva a `String` resultante tanto em `createdAt` quanto em `updatedAt`.

#### Construtor 2: Para carregar tarefas EXISTENTES do JSON

```java
    public Task(int id, String description, String status,
                String createdAt, String updatedAt) {
        this.id = id;
        this.description = description;
        this.status = status;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }

```

* **Utilidade**: Usado pelo reconstrutor de JSON (`JsonHandler`).
* **Lógica**: Quando lemos o arquivo `tasks.json`, a tarefa já possui histórico. Este construtor aceita esses dados antigos e recria o objeto na memória mantendo as datas e status originais intocados.

---

### 4. Métodos de Acesso (Getters e Setters)

```java
    public int getId()           { return id; }
    public String getDescription() { return description; }
    public String getStatus()    { return status; }
    public String getCreatedAt() { return createdAt; }
    public String getUpdatedAt() { return updatedAt; }

```

* **Getters**: Portas públicas de leitura rápida para retornar os valores protegidos por `private`.

```java
    public void setDescription(String description) {
        this.description = description;
        this.updatedAt = LocalDateTime.now().format(FORMATTER); 
    }

    public void setStatus(String status) {
        this.status = status;
        this.updatedAt = LocalDateTime.now().format(FORMATTER); 
    }

```

* **Setters**: Portas de escrita. Toda vez que a descrição ou o status mudam, a linha `this.updatedAt = LocalDateTime.now().format(FORMATTER);` intercepta a alteração e atualiza o marcador temporal automaticamente, garantindo a rastreabilidade do ciclo de vida da tarefa.

---

### 5. Mecanismo de Serialização JSON Manual

Como não utilizaremos bibliotecas externas (como Jackson ou Gson), a própria classe `Task` sabe se traduzir para o formato JSON usando manipulação nativa de Strings.

```java
    public String toJson() {
        return String.format(
                "  {\n" +
                "    \"id\": %d,\n" +
                "    \"description\": \"%s\",\n" +
                "    \"status\": \"%s\",\n" +
                "    \"createdAt\": \"%s\",\n" +
                "    \"updatedAt\": \"%s\"\n" +
                "  }",
                id,
                escaparJson(description), 
                status,
                createdAt,
                updatedAt
        );
    }

```

* **`String.format()`**: Funciona como um template. Os marcadores (`%d` para inteiros, `%s` para textos/Strings) são substituídos na ordem pelas variáveis passadas após a vírgula.
* **`\n` e `\"**`: Sequências de escape. O `\n` pula linha para deixar o arquivo JSON formatado e legível para humanos, enquanto `\"` permite colocar aspas dentro de uma String sem que o Java ache que a string acabou.

#### Tratamento de Segurança no JSON

```java
    private String escaparJson(String texto) {
        return texto
                .replace("\\", "\\\\")  
                .replace("\"", "\\\"")  
                .replace("\n", "\\n")   
                .replace("\r", "\\r");  
    }

```

* **Por que isso é necessário?** Se o usuário criar uma tarefa chamada: `Limpar o quarto antes de "jogar"`, as aspas em `"jogar"` quebrariam a sintaxe do arquivo JSON final. O método `.replace()` busca caracteres perigosos e adiciona barras de escape (`\"`), higienizando o texto e prevenindo erros na gravação e na leitura do arquivo.
* **Atenção na Ordem**: A substituição da barra invertida (`\\`) deve vir **sempre primeiro**, caso contrário ela escaparia as próprias barras inseridas pelos replaces seguintes.

---

### 6. Exibição Formatada no Terminal (`toString`)

```java
    @Override
    public String toString() {
        return String.format(
                "[%d] %-45s | Status: %-12s | Criado: %s | Atualizado: %s",
                id, description, status, createdAt, updatedAt
        );
    }

```

* **`@Override`**: Uma anotação que avisa ao compilador Java que estamos reescrevendo o método padrão `toString()` que todo objeto Java possui por padrão.
* **Alinhamento de colunas (`%-45s` e `%-12s`)**: O sinal de menos `-` alinha o texto à esquerda, e o número define o tamanho mínimo da coluna.
* Se a descrição tiver apenas 10 caracteres, o Java adicionará 35 espaços em branco logo após ela. Isso garante que as barras verticais (`|`) fiquem perfeitamente alinhadas no terminal como uma tabela, independentemente do tamanho do texto da tarefa.



---

Prontinho! O arquivo `progresso.md` está mapeado de ponta a ponta. Pode salvá-lo no seu diretório de estudos.

Quando terminar de digitar todo o código do arquivo `Task.java` e estiver pronto para avançar para o **Passo 3** do tutorial, me chame!