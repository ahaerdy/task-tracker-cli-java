# Fundamentos do Passo 2 — Criando o Modelo: `Task.java`

Nesta etapa, construímos o modelo de dados que serve como alicerce do projeto. A classe `Task` define o comportamento e a estrutura de uma tarefa na memória, além de possuir os mecanismos para se auto-converter em texto JSON e exibir-se de forma organizada no terminal.

## 1. Estrutura Inicial e a Constante de Formatação

```java
package tasktracker;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;  // Importa a classe DateTimeFormatter, responsável por 
                                            // definir e aplicar máscaras de leitura e escrita em datas

public class Task {

    // public:    Acessível por qualquer outra classe do projeto (ex: Main.java)
    // static:    Pertence à classe Task; existe apenas uma instância na memória para todo o app
    // final:     É uma constante; o valor não pode ser alterado após definido
    // FORMATTER: Nome da constante (em maiúsculas por convenção do Java)
    public static final DateTimeFormatter FORMATTER =
            // .ofPattern: Método que constrói o formatador baseado na máscara fornecida:
            // yyyy: Ano (4 dígitos) | MM: Mês (2 dígitos) | dd: Dia (2 dígitos)
            // HH: Hora 24h (2 dígitos) | mm: Minutos (2 dígitos) | ss: Segundos (2 dígitos)
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
}

```

* **`package tasktracker;`**: Define o escopo organizacional do arquivo.
* **`import`**: Importa a biblioteca de tempo do Java.
* **`public static final DateTimeFormatter FORMATTER`**: Cria uma constante universal acessível por todo o projeto. Ela define a "máscara" visual (`yyyy-MM-dd HH:mm:ss`) para que as datas fiquem fáceis de ler por humanos, impedindo o formato bruto do Java (ex: `2026-05-27T19:10:19`).

### Anatomia da Declaração:

```plaintext
[O que o Java executa] ───> DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
                                 │
                                 ▼ (Retorna um objeto formatador configurado)
                                 │
 [Onde ele armazena]   ───> FORMATTER
                                 │
                                 ▼ (Que é do tipo...)
                                 │
 [Tipo da variável]    ───> DateTimeFormatter
                                 │
                                 ▼ (Com as propriedades...)
                                 │
 [Modificadores]       ───> public static final
```

### 1.2. Testando a Estrutura: Main.java

Para garantir que a nossa constante de formatação está funcionando corretamente antes de avançarmos para as regras de negócio das tarefas, criamos uma classe de teste temporária. A `Main.java` servirá para validar o comportamento do `FORMATTER` acessando-o diretamente, sem a necessidade de instanciar a classe `Task`.

```java
package tasktracker; // Define que esta classe pertence ao mesmo pacote que a classe Task

public class Main {
    // Método principal: é a porta de entrada do programa (onde a execução começa)
    public static void main(String[] args) {
        
        // Task.FORMATTER: Acessa a constante estática diretamente pelo nome da classe "Task"
        // Sem o modificador 'static', seríamos obrigados a fazer 'new Task().FORMATTER'
        // System.out.println: Imprime no terminal a representação textual interna do objeto formatador
        System.out.println("Instância do Formatador: " + Task.FORMATTER);

        // java.time.LocalDateTime.now(): Captura a data e o horário exato do sistema neste instante
        // armazenando o resultado na variável 'agora' do tipo LocalDateTime
        java.time.LocalDateTime agora = java.time.LocalDateTime.now();

        // Imprime o objeto de data bruto no padrão ISO-8601 (ex: 2026-05-28T14:18:30.923079581)
        // Útil para contrastar a diferença entre a data nativa do Java e a versão que será formatada abaixo
        System.out.println("Data e Hora atuais no formato bruto: " + agora);

        // agora.format(...): Aplica a máscara da nossa constante sobre o objeto de data atual
        // O Java processa o LocalDateTime e devolve uma String organizada no padrão "yyyy-MM-dd HH:mm:ss"
        String dataFormatada = agora.format(Task.FORMATTER);

        // Exibe no terminal o texto final já devidamente formatado para o usuário
        System.out.println("Data e Hora atuais formatadas: " + dataFormatada);
    }
}

```

* `Task.FORMATTER`: Demonstra o poder do modificador `static`. A `Main` consegue ler a constante diretamente apontando para a classe dona do recurso.
* `System.out.println("... formato bruto: " + agora)`: Exibe o estado nativo do objeto de tempo do Java, expondo o separador padrão `T` e frações de segundos.
* `agora.format(...)`: O método recebe o molde imutável que criamos e lapida os dados brutos do relógio do sistema.

**Anatomia da Execução na Main:**

```text
                       ┌───> [Nativo ISO-8601] ──> 2026-05-28T14:18:30.923079581
                       │
[Relógio do Sistema] ──┼─> java.time.LocalDateTime.now() (Data/Hora Bruta)
                       │
                       └───> (Passa pelo filtro...)
                                     │
[Filtro de Destino]  ───────> Task.FORMATTER ("yyyy-MM-dd HH:mm:ss")
                                     │
                                     ▼ (Gera o resultado lapidado)
                                     │
[Saída Convertida]   ───────> "2026-05-28 14:18:30" (String pronta para o terminal)

```

### 1.3. Saída do Terminal (Output)

Ao compilar e executar o código acima no IntelliJ, o resultado exibido no seu console será o seguinte:

```text
/usr/lib/jvm/default-java/bin/java -javaagent:/snap/intellij-idea-community/762/lib/idea_rt.jar=40847 -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath /home/arthur/Downloads/Projetos/task-tracker/task-tracker/out/production/task-tracker tasktracker.Main

Instância do Formatador: Value(YearOfEra,4,19,EXCEEDS_PAD)'-'Value(MonthOfYear,2)'-'Value(DayOfMonth,2)' 'Value(HourOfDay,2)':'Value(MinuteOfHour,2)':'Value(SecondOfMinute,2)
Data e Hora atuais no formato bruto: 2026-05-28T14:18:30.923079581
Data e Hora atuais formatadas: 2026-05-28 14:18:30

Process finished with exit code 0

```

#### 1.3.1. Decodificando a Saída do Console

Quando executamos o `System.out.println(Task.FORMATTER);`, o Java expõe as entranhas do objeto. Em vez de uma string simples, ele imprime a **árvore de regras estruturais** que montamos através da máscara.

A saída no console é uma String gerada automaticamente pelo método `.toString()` interno do objeto `DateTimeFormatter`. Veja o que cada elemento significa na prática:

```text
Value(YearOfEra,4,19,EXCEEDS_PAD)'-'Value(MonthOfYear,2)'-'Value(DayOfMonth,2)' 'Value(HourOfDay,2)':'Value(MinuteOfHour,2)':'Value(SecondOfMinute,2)

```

* **`Value(YearOfEra,4,19,EXCEEDS_PAD)`** ──> Representa o padrão **`yyyy`** (Ano).
* *`YearOfEra`*: Indica que o cálculo adota a era atual (D.C.).
* *`4`*: Define o preenchimento visual padrão para 4 dígitos (ex: `2026`).
* *`19`*: É o limite máximo de caracteres suportado internamente pelo Java para este campo.
* *`EXCEEDS_PAD`*: Política de segurança. Se o ano ultrapassar 4 dígitos no futuro (ex: ano `12026`), o Java expandirá o tamanho automaticamente para não truncar a informação.


* **`'-'` , `' '` , `':'**` ──> São os **caracteres literais** isolados em aspas simples. Eles agem como separadores estáticos idênticos aos que digitamos na máscara.
* **`Value(MonthOfYear,2)`** ──> Representa o padrão **`MM`** (Mês com 2 dígitos fixos).
* **`Value(DayOfMonth,2)`** ──> Representa o padrão **`dd`** (Dia com 2 dígitos fixos).
* **`Value(HourOfDay,2)`** ──> Representa o padrão **`HH`** (Hora no formato de ciclo 24h, com 2 dígitos).
* **`Value(MinuteOfHour,2)`** ──> Representa o padrão **`mm`** (Minutos com 2 dígitos).
* **`Value(SecondOfMinute,2)`** ──> Representa o padrão **`ss`** (Segundos com 2 dígitos).

#### 1.3.2. Formato Bruto vs Formato Lapidado

* **`Data e Hora atuais no formato bruto: 2026-05-28T14:18:30.923079581`**
Esta linha exibe o comportamento nativo do Java (ISO-8601). O caractere `T` separa rigidamente a data do horário, e o valor termina com a precisão máxima de nanossegundos do sistema operacional. Embora ideal para persistência em bancos de dados, é um formato poluído para telas de linha de comando.
* **`Data e Hora atuais formatadas: 2026-05-28 14:18:30`**
Este é o output real do processamento do nosso `FORMATTER`. Toda a mecânica descrita na árvore de regras serviu de fôrma para capturar os dados brutos e limpá-los, gerando uma linha de texto perfeitamente legível e padronizada.

#### 1.3.3. Ciclo de Encerramento

* **`Process finished with exit code 0`**
A confirmação do ambiente de desenvolvimento de que a Máquina Virtual Java executou todas as instruções da classe `Main`, da primeira à última linha, e foi encerrada com sucesso total, sem disparar exceções (`Exception`) ou travar a execução do sistema.

> 💡 **Nota:** Repare que a primeira linha do output exibe a estrutura interna (o "esqueleto" técnico) que o Java criou para o `DateTimeFormatter`. Já a segunda linha exibe o resultado real do processamento, transformando dados complexos de tempo em texto perfeitamente legível.

---

## 2. Atributos Privados (Campos de Dados)

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

---

## 3. Sobrecarga de Construtores (Polimorfismo)

O Java permite definir mais de um construtor para a mesma classe, desde que eles recebam parâmetros diferentes. Isso se chama **Sobrecarga**.

### Construtor 1: Para NOVAS tarefas

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

### Construtor 2: Para carregar tarefas EXISTENTES do JSON

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

## 4. Métodos de Acesso (Getters e Setters)

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

## 5. Mecanismo de Serialização JSON Manual

Como não utilizaremos bibliotecas externas (como Jackson ou Gson), a própria classe `Task` deve traduzir PARA o formato JSON usando manipulação nativa de Strings.

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

### Tratamento de Segurança no JSON

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

### 5.1. Código de Implementação Mínimo para Teste

Para validar a serialização manual e o comportamento do método de escape sem a complexidade do restante do projeto, estruturamos uma versão enxuta da classe `Task` (contendo apenas os atributos simulados e os métodos do JSON) e uma classe `Main` preparada para testar um cenário real com caracteres especiais.

#### Classe Task.java (Módulos de Serialização)

```java
package tasktracker;

public class Task {
    // Atributos privados simulados para o teste de serialização
    private int id;
    private String description;
    private String status;
    private String createdAt;
    private String updatedAt;

    // Construtor minimalista para instanciar a tarefa diretamente no teste
    public Task(int id, String description, String status, String createdAt, String updatedAt) {
        this.id = id;
        this.description = description;
        this.status = status;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }

    // Método que traduz as propriedades do objeto para uma String formatada em JSON
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
                escaparJson(description), // Aplica a higienização contra quebras de sintaxe
                status,
                createdAt,
                updatedAt
        );
    }

    // Método de segurança que neutraliza caracteres que danificariam a estrutura do JSON
    private String escaparJson(String texto) {
        if (texto == null) return "";
        return texto
                .replace("\\", "\\\\")  // Substitui a barra invertida por duas (DEVE ser o primeiro)
                .replace("\"", "\\\"")  // Escapa aspas duplas internas para não fechar a string do JSON
                .replace("\n", "\\n")   // Transforma quebras de linha reais em representações textuais
                .replace("\r", "\\r");  // Transforma retornos de carro em representações textuais
    }
}

```

#### Classe Main.java (Invocação e Validação)

```java
package tasktracker;

public class Main {
    public static void main(String[] args) {
        System.out.println("=== TESTANDO SERIALIZAÇÃO MANUAL E HIGIENIZAÇÃO JSON ===");

        // Cenário de Teste: Descrição complexa contendo aspas e quebra de linha simulada (\n)
        // O texto original simula o usuário digitando: Estudar Java com "foco"
        //                                              Urgente!
        String descricaoComConflito = "Estudar Java com \"foco\"\nUrgente!";

        // Instancia uma tarefa com dados fictícios fixos e a descrição problemática
        Task tarefaTeste = new Task(
                1,
                descricaoComConflito,
                "todo",
                "2026-05-28 14:00:00",
                "2026-05-28 14:30:00"
        );

        // Dispara a conversão para texto JSON
        String jsonResultado = tarefaTeste.toJson();

        // Imprime o resultado final no terminal para checagem visual
        System.out.println("\nJSON Gerado com Sucesso:");
        System.out.println(jsonResultado);
        
        System.out.println("\n=======================================================");
    }
}

```

### 5.2. Anatomia do Fluxo de Higienização

Quando a `Main` executa o método `toJson()`, a String de descrição passa por uma linha de montagem de substituições. É crucial entender a transformação do texto cru para o texto estéril:

```text
[Texto Original na Main] ──> Estudar Java com "foco"\nUrgente!
                                      │
                                      ▼ (.replace("\"", "\\\""))
[Passo 1: Escapa Aspas]  ──> Estudar Java com \"foco\"\nUrgente!
                                      │
                                      ▼ (.replace("\n", "\\n"))
[Passo 2: Escapa Quebra] ──> Estudar Java com \"foco\"\\nUrgente!
                                      │
                                      ▼ (Injetado dentro do molde String.format)
[Resultado no JSON]      ──> "description": "Estudar Java com \"foco\"\\nUrgente!"

```

### 5.3. Saída do Terminal (Output Esperado)

Ao executar a classe `Main` no IntelliJ, a saída reproduzirá perfeitamente a estrutura de chaves e espaçamentos de um arquivo `.json` válido:

```text
=== TESTANDO SERIALIZAÇÃO MANUAL E HIGIENIZAÇÃO JSON ===

JSON Gerado com Sucesso:
  {
    "id": 1,
    "description": "Estudar Java com \"foco\"\nUrgente!",
    "status": "todo",
    "createdAt": "2026-05-28 14:00:00",
    "updatedAt": "2026-05-28 14:30:00"
  }

=======================================================
Process finished with exit code 0

```

#### Análise Crítica da Saída:

1. **Validação das Aspas:** Note que na linha do `"description"`, as aspas que cercam a palavra `\"foco\"` aparecem acompanhadas da barra invertida. Isso garante que qualquer leitor de JSON posterior compreenda que aquelas aspas pertencem ao texto, e não ao fechamento do campo do JSON.
2. **Preservação do Layout:** Os recuos de dois espaços (`  `) e as quebras de linha (`\n`) configurados dentro do `String.format()` montaram um bloco identado limpo, provando que a serialização manual cumpre com rigor a estética padrão do formato de dados JSON.

---

## 6. Exibição Formatada no Terminal (`toString`)

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

### 6.1. Código de Implementação Mínimo para Teste do Layout

Para testar o alinhamento de colunas (`%-45s` e `%-12s`), utilizaremos uma versão enxuta da classe `Task` focada no método de formatação, e uma classe `Main` que instanciará tarefas com descrições de tamanhos completamente diferentes (uma muito curta e outra longa) para provar que o alinhamento das barras verticais (`|`) permanece intacto.

#### Classe Task.java (Módulo de Visualização)

```java
package tasktracker;

public class Task {
    // Atributos privados que compõem a linha do terminal
    private int id;
    private String description;
    private String status;
    private String createdAt;
    private String updatedAt;

    // Construtor minimalista para alimentar o teste
    public Task(int id, String description, String status, String createdAt, String updatedAt) {
        this.id = id;
        this.description = description;
        this.status = status;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }

    // [MÉTODO DO ITEM 6]: Sobrescreve a exibição padrão do objeto para criar o layout de tabela
    @Override
    public String toString() {
        return String.format(
                "[%d] %-45s | Status: %-12s | Criado: %s | Atualizado: %s",
                id, description, status, createdAt, updatedAt
        );
    }
}

```

#### Classe Main.java (Validação do Alinhamento Visual)

```java
package tasktracker;

public class Main {
    public static void main(String[] args) {
        System.out.println("=== TESTANDO ALINHAMENTO DE COLUNAS NO TERMINAL (toString) ===\n");

        // Tarefa 1: Descrição curtíssima (11 caracteres). O Java deve preencher o restante com 34 espaços.
        Task t1 = new Task(1, "Comprar café", "todo", "2026-05-28 15:00:00", "2026-05-28 15:00:00");

        // Tarefa 2: Descrição longa (41 caracteres). O Java deve preencher com apenas 4 espaços.
        Task t2 = new Task(2, "Estudar estrutura de dados e coleções em Java", "in-progress", "2026-05-28 15:10:00", "2026-05-28 15:40:00");

        // Tarefa 3: ID maior e descrição média (24 caracteres). Testando a elasticidade do layout.
        Task t3 = new Task(105, "Configurar Docker Compose", "done", "2026-05-28 10:00:00", "2026-05-28 11:15:00");

        // Imprime as tarefas simulando uma listagem do CLI
        System.out.println(t1);
        System.out.println(t2);
        System.out.println(t3);

        System.out.println("\n==============================================================");
    }
}

```

---

### 6.2. Anatomia do Alinhamento (`String.format`)

O segredo do método está nos sinalizadores de largura estática. O Java calcula o tamanho do texto dinamicamente e preenche a diferença com caracteres invisíveis de paginação:

```text
 Máscara:  "[%d] %-45s | Status: %-12s | ..."
             │     │   │           │
             │     │   │           └──────> Reserva 12 espaços alinhados à esquerda
             │     │   └──────────────────> Caractere delimitador (Barra estática)
             │     └──────────────────────> Reserva 45 espaços alinhados à esquerda
             └────────────────────────────> ID dinâmico em colchetes

 Na prática para a Tarefa 1:
 "Comprar café" (11 letras) + 34 espaços em branco gerados automaticamente = Coluna de 45 caracteres.

```

---

### 6.3. Saída do Terminal (Output Esperado)

Ao executar a classe `Main` no IntelliJ, veja como o caractere pipe (`|`) se comporta como uma linha divisória perfeita de planilha:

```text
=== TESTANDO ALINHAMENTO DE COLUNAS NO TERMINAL (toString) ===

[1] Comprar café                                   | Status: todo         | Criado: 2026-05-28 15:00:00 | Atualizado: 2026-05-28 15:00:00
[2] Estudar estrutura de dados e coleções em Java  | Status: in-progress  | Criado: 2026-05-28 15:10:00 | Atualizado: 2026-05-28 15:40:00
[105] Configurar Docker Compose                    | Status: done         | Criado: 2026-05-28 10:00:00 | Atualizado: 2026-05-28 11:15:00

==============================================================
Process finished with exit code 0

```

#### Análise Crítica do Output:

1. **O Efeito do Hífen (`-`)**: Sem o sinal de menos, o Java alinharia o texto à direita (encostando o título da tarefa na barra `|` e deixando os espaços vazios do lado esquerdo). O uso do `%-45s` garante a legibilidade natural de leitura ocidental (da esquerda para a direita).
2. **Independência de ID**: Mesmo quando o ID salta de 1 dígito (`1`) para 3 dígitos (`105`) (empurrando o colchete de fechamento um pouco para o lado), a coluna da descrição absorve o impacto e mantém a barra vertical perfeitamente reta no mesmo alinhamento das linhas superiores.

---

