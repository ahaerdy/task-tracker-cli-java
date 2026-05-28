## Progresso - Task Tracker CLI (Java)

### Fase: Passo 2 — Criando o Modelo: `Task.java`

Nesta etapa, construímos o modelo de dados que serve como alicerce do projeto. A classe `Task` define o comportamento e a estrutura de uma tarefa na memória, além de possuir os mecanismos para se auto-converter em texto JSON e exibir-se de forma organizada no terminal.

### 1. Estrutura Inicial e a Constante de Formatação

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

#### Anatomia da Declaração:

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

Perfeito, Arthur. Entendi o contexto. Como se trata de um tutorial passo a passo, a transição para o código de teste (`Main.java`) precisa ser fluida, mostrando primeiro a aplicação prática da teoria e, logo em seguida, o impacto visual (o output).

Aqui está a continuação estruturada para o seu tutorial, mantendo exatamente o mesmo padrão didático, os comentários inline e o diagrama de fluxo que você usou no Passo 2:

### 2. Testando a Eestrutura: Main.java

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
        
        // agora.format(...): Aplica a máscara da nossa constante sobre o objeto de data atual
        // O Java processa o LocalDateTime e devolve uma String organizada no padrão "yyyy-MM-dd HH:mm:ss"
        String dataFormatada = agora.format(Task.FORMATTER);

        // Exibe no terminal o texto final já devidamente formatado para o usuário
        System.out.println("Data e Hora atual formatada: " + dataFormatada);
    }
}

```

`Task.FORMATTER`: Demonstra o poder do modificador `static`. A `Main` consegue ler a constante diretamente apontando para a classe dona do recurso.
`agora.format(...)`: O método recebe o molde imutável que criamos e lapida os dados brutos do relógio do sistema.

**Anatomia da Execução na Main:**

```text
[Relógio do Sistema] ───> java.time.LocalDateTime.now() (Data/Hora Bruta)
                                      │
                                      ▼ (Passa pelo filtro...)
                                      │
[Filtro de Destino]  ───> Task.FORMATTER ("yyyy-MM-dd HH:mm:ss")
                                      │
                                      ▼ (Gera o resultado lapidado)
                                      │
[Saída Convertida]   ───> "2026-05-28 13:53:32" (String pronta para o terminal)

```

### 3. Saída do Terminal (Output)

Ao compilar e executar o código acima, o resultado exibido no seu console será o seguinte:

```text
Instância do Formatador: ParseCaseSensitive(Strict).Value(Year,4).Literal('-').Value(MonthOfYear,2).Literal('-').Value(DayOfMonth,2).Literal(' ').Value(HourOfDay,2).Literal(':').Value(MinuteOfHour,2).Literal(':').Value(SecondOfMinute,2)
Data e Hora atual formatada: 2026-05-28 13:53:32

```

> 💡 **Nota:** Repare que a primeira linha do output exibe a estrutura interna (o "esqueleto" técnico) que o Java criou para o `DateTimeFormatter`. Já a segunda linha exibe o resultado real do processamento, transformando dados complexos de tempo em texto perfeitamente legível.

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

Excelente! O Passo 3 é crucial porque introduz a cultura de **testes laboratoriais** no desenvolvimento de software. Em vez de construir o sistema inteiro às cegas para descobrir bugs só no final, criamos uma classe isolada (`Main`) para validar se a nossa fundação (`Task.java`) responde exatamente como esperado.

Aqui está o arquivo `progresso.md` completo, ultra detalhado e didático para você documentar o seu avanço nesta etapa:

---

### Fase: Passo 3 — Testando o Modelo `Task.java`

Nesta etapa, criamos um ponto de entrada temporário para o programa através da classe `Main.java`. O objetivo é executar testes unitários manuais no console para estressar os métodos da classe `Task` (instanciação, encapsulamento, atualização automática de timestamps e escape de caracteres no JSON).

---

### 1. O Ponto de Entrada: `public static void main`

Todo programa Java independente precisa de uma porta de entrada para saber por onde começar a execução.

```java
package tasktracker;

public class Main {
    public static void main(String[] args) {

```

* **`public static void main(String[] args)`**: É a assinatura padrão do método de inicialização do Java.
* `public`: Permite que a máquina virtual Java (JVM) encontre e execute o método de fora.
* `static`: Permite que o método seja chamado sem que a JVM precise criar um objeto da classe `Main` primeiro.
* `void`: Significa que o método realiza tarefas, mas não retorna nenhum valor ao final.
* `String[] args`: Um array de textos que permite passar parâmetros via terminal para o programa (muito importante mais para a frente no projeto).



---

### 2. Anatomia dos Blocos de Teste

O script executa 5 cenários controlados para avaliar o comportamento do nosso modelo. Vamos destrinchar o que cada um valida nos bastidores:

#### TESTE 1: Instanciação e Leitura de Atributos (Getters e `toString`)

```java
Task t1 = new Task(1, "Comprar leite");
System.out.println(t1);                    
System.out.println("ID: " + t1.getId());

```

* **O que acontece:** A palavra-chave `new` invoca o nosso **primeiro construtor**, injetando o ID e a descrição. O status padrão `"todo"` e os marcadores de tempo idênticos são calculados na hora.
* **Mecanismo Oculto (`System.out.println(t1);`)**: Quando passamos um objeto direto para o `println`, o Java invoca automaticamente o método `.toString()` daquele objeto. Como reescrevemos esse método usando a etiqueta `@Override`, ele imprime a linha tabular com as colunas perfeitamente alinhadas pelo operador `%-45s`.
* **Validação do Encapsulamento**: As linhas seguintes chamam os métodos `getId()`, `getDescription()`, etc. Isso prova que as portas de leitura (`getters`) estão funcionando para expor os dados que foram protegidos com o modificador `private`.

#### TESTE 2 e 3: Mutação de Dados e Ciclo de Vida do Timestamp (`setters`)

```java
t1.setDescription("Comprar leite e ovos");
System.out.println(t1);
t1.setStatus("in-progress");

```

* **O que acontece:** O teste chama as portas de escrita (`setters`).
* **Comportamento Esperado no Relógio:** No console, a data/hora de criação (`Criado:`) deve permanecer idêntica à do Teste 1, mas o campo (`Atualizado:`) precisa mudar para o segundo exato em que a alteração ocorreu. Isso valida que a linha interna dos nossos métodos `set` (`LocalDateTime.now().format(FORMATTER)`) está disparando corretamente.

#### TESTE 4: Estrutura da Serialização JSON Manual

```java
System.out.println(t1.toJson());

```

* **O que acontece:** Invoca o método `.toJson()` criado no Passo 2.
* **Mecanismo Oculto**: O método usa `String.format()` injetando as variáveis dentro das quebras de linha (`\n`) e aspas escapadas (`\"`). O resultado esperado é um bloco textual perfeitamente identado que simula a sintaxe de um dicionário JSON chave/valor.

#### TESTE 5: Validação da Camada de Segurança (Método `escaparJson`)

```java
Task t2 = new Task(2, "Ler o livro \"Clean Code\"");
System.out.println(t2.toJson());

```

* **O que acontece:** Esse é o teste de estresse mais inteligente do script. Ele insere uma String que já contém aspas internas.
* **Mecanismo Oculto**: Ao chamar o método privado `.escaparJson(description)` dentro de `toJson()`, o Java executa a corrente de métodos `.replace()`.
* **O Resultado Visual**: No console, o texto impresso deve aparecer como `"description": "Ler o livro \"Clean Code\""`. As barras invertidas extras provam que o JSON final não será corrompido, pois o interpretador saberá diferenciar as aspas do texto das aspas que delimitam o campo.

---

### 3. Diagnóstico e Resultados do Experimento

Ao rodar o comando **Shift + F10** (no IntelliJ ou NetBeans) ou rodar via terminal, o console despeja o log estruturado.

* Se as colunas saírem alinhadas, o `@Override toString()` passou.
* Se os segundos mudarem dinamicamente nos testes 2 e 3, a reatividade do `updatedAt` passou.
* Se as aspas do livro *Clean Code* vierem acompanhadas de uma barra invertida antes delas, o método `escaparJson()` passou.

> **Status do Passo 3:** Modelo homologado e pronto para ser integrado às classes de persistência de arquivos!

