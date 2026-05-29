# Tutorial: Task Tracker CLI em Java
### Roadmap.sh Challenge — Guia Passo a Passo (IntelliJ IDEA)

> **Objetivo:** Construir um aplicativo de linha de comando (CLI) em Java puro para gerenciar tarefas, sem bibliotecas externas.
> Você vai aprender: classes, getters/setters, manipulação de arquivos, expressões regulares, streams e estrutura de projeto Java.

---

## Índice

1. [Configurando o projeto no IntelliJ](#passo-1)
2. [Criando o modelo: `Task.java`](#passo-2)
3. [Testando o modelo Task](#passo-3)
4. [Criando o leitor/escritor JSON: `JsonHandler.java`](#passo-4)
5. [Testando a persistência JSON](#passo-5)
6. [Criando a lógica de negócio: `TaskManager.java`](#passo-6)
7. [Testando o TaskManager](#passo-7)
8. [Criando o ponto de entrada: `Main.java`](#passo-8)
9. [Testes completos via terminal](#passo-9)
10. [Gerando o JAR executável](#passo-10)

---

## Passo 1 — Configurando o Projeto no IntelliJ {#passo-1}

### 1.1 Criando o projeto

1. Abra o **IntelliJ IDEA**
2. Clique em **New Project**
3. No painel esquerdo, selecione **Java**
4. Configure assim:
   - **Name:** `task-tracker`
   - **Location:** escolha uma pasta de sua preferência
   - **Language:** Java
   - **Build system:** IntelliJ (não Maven/Gradle, por simplicidade)
   - **JDK:** selecione Java 11 ou superior (qualquer versão ≥ 11 serve)
5. **Desmarque** "Add sample code"
6. Clique em **Create**

### 1.2 Criando o pacote

1. No painel **Project** (esquerda), expanda `task-tracker > src`
2. Clique com o botão direito em `src` → **New → Package**
3. Digite: `tasktracker`
4. Pressione Enter

Sua estrutura deve estar assim:
```
task-tracker/
└── src/
    └── tasktracker/   ← pacote criado agora
```

---

## Passo 2 — Criando o Modelo: `Task.java` {#passo-2}

O modelo `Task` representa uma tarefa. É o alicerce de tudo — começamos por ele.

> 🧪 **Fundamentos:**
> Antes de aplicar o código abaixo, você pode conferir a análise detalhada da sintaxe, testes isolados e a anatomia das engrenagens deste módulo no guia de [Fundamentos do Passo 2](./fundamentos/passo_2).

### 2.1 Criando o arquivo

1. Clique com o botão direito no pacote `tasktracker` → **New → Java Class**
2. Digite: `Task`
3. Pressione Enter

### 2.2 Digite o seguinte código

```java
package tasktracker;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * Modelo de dados de uma tarefa no Task Tracker.
 *
 * Cada tarefa tem:
 *   - id          → número único que identifica a tarefa
 *   - description → texto descrevendo o que precisa ser feito
 *   - status      → estado atual: "todo", "in-progress" ou "done"
 *   - createdAt   → data/hora de criação (nunca muda)
 *   - updatedAt   → data/hora da última alteração (atualizada automaticamente)
 */
public class Task {

    // Formato de data e hora usado em todo o projeto: "2024-01-15 09:30:00"
    public static final DateTimeFormatter FORMATTER =
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    // ─── Atributos privados ────────────────────────────────────────────────────
    private int id;
    private String description;
    private String status;      // Valores válidos: "todo" | "in-progress" | "done"
    private String createdAt;   // Data de criação (imutável após criação)
    private String updatedAt;   // Data da última modificação

    // ─── Construtor para NOVAS tarefas ─────────────────────────────────────────
    /**
     * Cria uma tarefa nova com status inicial "todo".
     * Os timestamps são preenchidos automaticamente com o horário atual.
     *
     * @param id         ID único gerado pelo TaskManager
     * @param description Texto que descreve a tarefa
     */
    public Task(int id, String description) {
        this.id = id;
        this.description = description;
        this.status = "todo";                                   // status padrão ao criar
        String agora = LocalDateTime.now().format(FORMATTER);  // momento atual formatado
        this.createdAt = agora;
        this.updatedAt = agora;
    }

    // ─── Construtor para carregar tarefas EXISTENTES do JSON ───────────────────
    /**
     * Usado pelo JsonHandler ao ler o arquivo tasks.json.
     * Restaura todos os campos exatamente como estavam salvos.
     *
     * @param id          ID salvo no arquivo
     * @param description Descrição salva
     * @param status      Status salvo
     * @param createdAt   Timestamp de criação salvo
     * @param updatedAt   Timestamp de atualização salvo
     */
    public Task(int id, String description, String status,
                String createdAt, String updatedAt) {
        this.id = id;
        this.description = description;
        this.status = status;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }

    // ─── Getters (leitura dos atributos privados) ──────────────────────────────
    public int getId()           { return id; }
    public String getDescription() { return description; }
    public String getStatus()    { return status; }
    public String getCreatedAt() { return createdAt; }
    public String getUpdatedAt() { return updatedAt; }

    // ─── Setters (modificam o atributo E atualizam o timestamp) ───────────────

    /**
     * Atualiza a descrição da tarefa.
     * updatedAt é renovado automaticamente para "agora".
     */
    public void setDescription(String description) {
        this.description = description;
        this.updatedAt = LocalDateTime.now().format(FORMATTER); // registra a mudança
    }

    /**
     * Muda o status da tarefa (ex: "todo" → "in-progress").
     * updatedAt é renovado automaticamente para "agora".
     */
    public void setStatus(String status) {
        this.status = status;
        this.updatedAt = LocalDateTime.now().format(FORMATTER); // registra a mudança
    }

    // ─── Serialização JSON ─────────────────────────────────────────────────────

    /**
     * Converte a tarefa em um fragmento JSON pronto para ser gravado no arquivo.
     *
     * Exemplo de saída:
     *   {
     *     "id": 1,
     *     "description": "Comprar leite",
     *     "status": "todo",
     *     "createdAt": "2024-01-15 09:00:00",
     *     "updatedAt": "2024-01-15 09:00:00"
     *   }
     */
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
                escaparJson(description), // escapa caracteres especiais
                status,
                createdAt,
                updatedAt
        );
    }

    /**
     * Escapa caracteres que têm significado especial dentro de strings JSON.
     * Sem isso, uma descrição como: ela disse "olá"
     * quebraria o JSON ao virar:   "description": "ela disse "olá""  ← inválido!
     *
     * @param texto Texto original da descrição
     * @return Texto seguro para uso dentro de uma string JSON
     */
    private String escaparJson(String texto) {
        return texto
                .replace("\\", "\\\\")  // \ → \\   (barra invertida, DEVE ser primeiro)
                .replace("\"", "\\\"")  // " → \"   (aspas duplas)
                .replace("\n", "\\n")   // newline → \n  (nova linha)
                .replace("\r", "\\r");  // carriage return → \r
    }

    // ─── toString para exibição no terminal ────────────────────────────────────

    /**
     * Formata a tarefa em uma linha legível para o terminal.
     *
     * Exemplo:
     *   [1] Comprar leite                           | Status: todo         | Criado: 2024-01-15 09:00:00 | Atualizado: 2024-01-15 09:00:00
     */
    @Override
    public String toString() {
        return String.format(
                "[%d] %-45s | Status: %-12s | Criado: %s | Atualizado: %s",
                id, description, status, createdAt, updatedAt
        );
    }
}
```

### 2.3 Salvando

Pressione **Ctrl+S** (ou Cmd+S no Mac). O IntelliJ compilará automaticamente.

> 💡 **Conceitos aprendidos neste passo:**
> - **Classe e atributos privados** — encapsulamento em Java
> - **Construtores sobrecarregados** — dois construtores com assinaturas diferentes
> - **Getters e Setters** — padrão JavaBean para acessar atributos privados
> - **`LocalDateTime`** — classe Java para trabalhar com data/hora
> - **`String.format()`** — formatação de strings com `%d`, `%s`, `%-45s`
> - **`@Override`** — sobrescreve o método `toString()` da classe `Object`

---

## Passo 3 — Testando o Modelo Task {#passo-3}

Antes de avançar, vamos verificar se `Task.java` funciona corretamente.

### 3.1 Criando Main.java temporário para testes

1. Clique com o botão direito em `tasktracker` → **New → Java Class**
2. Digite: `Main`
3. Pressione Enter
4. Cole o código de teste abaixo:

```java
package tasktracker;

public class Main {
    public static void main(String[] args) {

        // ── TESTE 1: Criar uma nova tarefa ──────────────────────────────
        System.out.println("=== TESTE 1: Criando tarefa nova ===");
        Task t1 = new Task(1, "Comprar leite");
        System.out.println(t1);                    // usa o toString()
        System.out.println("ID: "          + t1.getId());
        System.out.println("Descrição: "   + t1.getDescription());
        System.out.println("Status: "      + t1.getStatus());
        System.out.println("Criado em: "   + t1.getCreatedAt());
        System.out.println("Atualizado em: " + t1.getUpdatedAt());

        // ── TESTE 2: Alterar descrição ──────────────────────────────────
        System.out.println("\n=== TESTE 2: Atualizando descrição ===");
        t1.setDescription("Comprar leite e ovos");
        System.out.println(t1);

        // ── TESTE 3: Alterar status ─────────────────────────────────────
        System.out.println("\n=== TESTE 3: Mudando status ===");
        t1.setStatus("in-progress");
        System.out.println(t1);

        // ── TESTE 4: Serialização JSON ──────────────────────────────────
        System.out.println("\n=== TESTE 4: JSON gerado ===");
        System.out.println(t1.toJson());

        // ── TESTE 5: Descrição com aspas (escape) ───────────────────────
        System.out.println("\n=== TESTE 5: Descrição com aspas (teste de escape) ===");
        Task t2 = new Task(2, "Ler o livro \"Clean Code\"");
        System.out.println(t2.toJson());
    }
}
```

### 3.2 Executando

Clique com o botão direito dentro do arquivo `Main.java` → **Run 'Main.main()'**

(Ou pressione **Shift+F10**)

### 3.3 Resultado esperado no console

```
=== TESTE 1: Criando tarefa nova ===
[1] Comprar leite                              | Status: todo         | Criado: 2024-01-15 09:10:00 | Atualizado: 2024-01-15 09:10:00
ID: 1
Descrição: Comprar leite
Status: todo
Criado em: 2024-01-15 09:10:00
Atualizado em: 2024-01-15 09:10:00

=== TESTE 2: Atualizando descrição ===
[1] Comprar leite e ovos                       | Status: todo         | Criado: 2024-01-15 09:10:00 | Atualizado: 2024-01-15 09:10:01

=== TESTE 3: Mudando status ===
[1] Comprar leite e ovos                       | Status: in-progress  | Criado: ...

=== TESTE 4: JSON gerado ===
  {
    "id": 1,
    "description": "Comprar leite e ovos",
    "status": "in-progress",
    "createdAt": "2024-01-15 09:10:00",
    "updatedAt": "2024-01-15 09:10:01"
  }

=== TESTE 5: Descrição com aspas (teste de escape) ===
  {
    "id": 2,
    "description": "Ler o livro \"Clean Code\"",
    ...
  }
```

---

## Passo 4 — Criando o Leitor/Escritor JSON: `JsonHandler.java` {#passo-4}

Agora criamos a classe responsável por **ler e salvar** as tarefas no arquivo `tasks.json`.
Como não podemos usar bibliotecas externas, vamos usar **expressões regulares (regex)** para fazer o parsing do JSON manualmente.

> 🧪 **Fundamentos:**
> Antes de aplicar o código abaixo, você pode conferir a análise detalhada da sintaxe, testes isolados e a anatomia das engrenagens deste módulo no guia de [Fundamentos do Passo 4](./fundamentos/passo_4).

### 4.1 Criando o arquivo

1. Clique com o botão direito em `tasktracker` → **New → Java Class**
2. Digite: `JsonHandler`
3. Pressione Enter

### 4.2 Digite o código completo

```java
package tasktracker;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.*;
import java.util.regex.*;

/**
 * Responsável por toda a persistência de dados do Task Tracker.
 *
 * Lê e grava o arquivo "tasks.json" na pasta atual de execução.
 * Não usa nenhuma biblioteca externa — parsing feito com Regex (java.util.regex).
 *
 * Estrutura do arquivo JSON:
 * [
 *   {
 *     "id": 1,
 *     "description": "...",
 *     "status": "todo",
 *     "createdAt": "...",
 *     "updatedAt": "..."
 *   },
 *   ...
 * ]
 */
public class JsonHandler {

    // Nome/caminho do arquivo onde as tarefas são persistidas.
    // O arquivo é criado na pasta onde o programa é executado.
    private static final String CAMINHO_ARQUIVO = "tasks.json";

    // ─── Leitura ──────────────────────────────────────────────────────────────

    /**
     * Carrega todas as tarefas salvas no arquivo JSON.
     *
     * Fluxo:
     *  1. Se o arquivo não existir → retorna lista vazia (não é erro)
     *  2. Lê o conteúdo completo do arquivo como string
     *  3. Usa regex para encontrar cada bloco { ... } no JSON
     *  4. Converte cada bloco em um objeto Task e adiciona à lista
     *
     * @return Lista de tarefas (pode estar vazia, nunca null)
     */
    public static List<Task> loadTasks() {
        List<Task> tarefas = new ArrayList<>();
        File arquivo = new File(CAMINHO_ARQUIVO);

        // Se o arquivo ainda não existe, é a primeira execução — lista vazia é normal
        if (!arquivo.exists()) {
            return tarefas;
        }

        try {
            // Lê todo o conteúdo do arquivo de uma vez como String UTF-8
            String conteudo = new String(
                    Files.readAllBytes(Paths.get(CAMINHO_ARQUIVO)),
                    StandardCharsets.UTF_8
            );

            /*
             * Regex para capturar cada objeto JSON individual.
             *
             * Pattern:  \{[^{}]+\}
             *
             * Explicação caractere a caractere:
             *   \{      → abre chave literal {  (precisa de escape pois { tem significado em regex)
             *   [^{}]+  → um ou mais caracteres que NÃO sejam { ou }
             *             (isso impede capturar objetos aninhados acidentalmente)
             *   \}      → fecha chave literal }
             *
             * Pattern.DOTALL faz o "." do regex casar com "\n" também,
             * necessário porque o JSON tem múltiplas linhas por objeto.
             */
            Pattern padraoBlocoJson = Pattern.compile("\\{[^{}]+\\}", Pattern.DOTALL);
            Matcher matcher = padraoBlocoJson.matcher(conteudo);

            // Para cada bloco { ... } encontrado no arquivo:
            while (matcher.find()) {
                String blocoJson = matcher.group(); // ex: { "id": 1, "description": "...", ... }
                Task tarefa = parsearTarefa(blocoJson);
                if (tarefa != null) {
                    tarefas.add(tarefa);
                }
            }

        } catch (IOException e) {
            System.err.println("Erro ao ler o arquivo " + CAMINHO_ARQUIVO + ": " + e.getMessage());
        }

        return tarefas;
    }

    // ─── Escrita ──────────────────────────────────────────────────────────────

    /**
     * Salva a lista completa de tarefas no arquivo JSON.
     * Substitui completamente o conteúdo anterior do arquivo.
     * Se o arquivo não existir, ele é criado automaticamente.
     *
     * @param tarefas Lista de tarefas a serem persistidas
     */
    public static void saveTasks(List<Task> tarefas) {
        StringBuilder sb = new StringBuilder();
        sb.append("[\n");  // abre o array JSON

        // Adiciona cada tarefa serializada
        for (int i = 0; i < tarefas.size(); i++) {
            sb.append(tarefas.get(i).toJson());  // chama o toJson() que fizemos em Task

            // Entre objetos JSON colocamos vírgula — mas NÃO após o último
            if (i < tarefas.size() - 1) {
                sb.append(",\n");
            }
        }

        sb.append("\n]");  // fecha o array JSON

        try {
            // Files.write cria o arquivo se não existir, ou sobrescreve se existir
            Files.write(
                    Paths.get(CAMINHO_ARQUIVO),
                    sb.toString().getBytes(StandardCharsets.UTF_8)
            );
        } catch (IOException e) {
            System.err.println("Erro ao salvar o arquivo " + CAMINHO_ARQUIVO + ": " + e.getMessage());
        }
    }

    // ─── Parsing interno ──────────────────────────────────────────────────────

    /**
     * Converte um fragmento de JSON (um objeto Task) em um objeto Java Task.
     *
     * Exemplo de entrada:
     *   {
     *     "id": 1,
     *     "description": "Comprar leite",
     *     "status": "todo",
     *     "createdAt": "2024-01-15 09:00:00",
     *     "updatedAt": "2024-01-15 09:00:00"
     *   }
     *
     * @param json Fragmento JSON de uma única tarefa
     * @return Objeto Task, ou null se o parsing falhar
     */
    private static Task parsearTarefa(String json) {
        try {
            // Extrai cada campo usando o método auxiliar extrairValor()
            int id                  = Integer.parseInt(extrairValor(json, "id"));
            String descricao        = extrairValor(json, "description");
            String status           = extrairValor(json, "status");
            String criadoEm        = extrairValor(json, "createdAt");
            String atualizadoEm   = extrairValor(json, "updatedAt");

            // Usa o construtor de 5 parâmetros (para tarefas existentes)
            return new Task(id, descricao, status, criadoEm, atualizadoEm);

        } catch (NumberFormatException e) {
            System.err.println("Erro: ID inválido no JSON — " + e.getMessage());
            return null;
        } catch (Exception e) {
            System.err.println("Erro ao interpretar tarefa do JSON: " + e.getMessage());
            return null;
        }
    }

    /**
     * Extrai o valor de um campo específico de um fragmento JSON.
     *
     * Suporta dois tipos de valor:
     *   - String:  "chave": "valor aqui"
     *   - Número:  "chave": 42
     *
     * Para strings, também desfaz os escapes aplicados pelo Task.escaparJson().
     *
     * @param json  Fragmento JSON do qual extrair
     * @param chave Nome do campo a extrair (ex: "description", "id")
     * @return Valor do campo como String, ou "" se não encontrado
     */
    private static String extrairValor(String json, String chave) {

        // ── Caso 1: valor do tipo String ───────────────────────────────────────
        /*
         * Regex:  "chave"\s*:\s*"((?:[^"\\]|\\.)*)"
         *
         * Decomposição:
         *   "chave"         → o nome do campo entre aspas (literal)
         *   \s*:\s*         → dois-pontos com zero ou mais espaços em volta
         *   "               → abre a string JSON
         *   (               → início do grupo de captura (o valor que queremos)
         *     (?:           → grupo não-capturante (não aparece no result)
         *       [^"\\]      → qualquer caractere exceto aspas e barra invertida
         *       |           → OU
         *       \\.         → barra invertida seguida de qualquer caractere (é um escape)
         *     )*            → repete zero ou mais vezes
         *   )               → fim do grupo de captura
         *   "               → fecha a string JSON
         *
         * Isso captura strings que podem conter \" e \\ no meio sem quebrar.
         */
        Pattern padraoString = Pattern.compile(
                "\"" + chave + "\"\\s*:\\s*\"((?:[^\"\\\\]|\\\\.)*)\""
        );
        Matcher m = padraoString.matcher(json);
        if (m.find()) {
            // Desfaz os escapes que foram aplicados ao salvar
            return m.group(1)
                    .replace("\\\"", "\"")   // \" → "
                    .replace("\\\\", "\\")   // \\ → \   (DEVE ser após o anterior)
                    .replace("\\n", "\n")    // \n → newline
                    .replace("\\r", "\r");   // \r → carriage return
        }

        // ── Caso 2: valor do tipo número inteiro ───────────────────────────────
        /*
         * Regex:  "chave"\s*:\s*(\d+)
         *   \d+   → um ou mais dígitos (0-9)
         */
        Pattern padraoNumero = Pattern.compile(
                "\"" + chave + "\"\\s*:\\s*(\\d+)"
        );
        m = padraoNumero.matcher(json);
        if (m.find()) {
            return m.group(1); // retorna o número como String (convertemos no chamador)
        }

        return ""; // campo não encontrado
    }
}
```

> 💡 **Conceitos aprendidos neste passo:**
> - **`java.nio.file.Files`** — API moderna do Java para ler/escrever arquivos
> - **`Pattern` e `Matcher`** — expressões regulares em Java
> - **`Pattern.DOTALL`** — flag que faz `.` casar com `\n`
> - **`StringBuilder`** — montagem eficiente de strings longas
> - **`StandardCharsets.UTF_8`** — garantia de codificação correta
> - **Regex de captura** — grupos `()` e grupos não-capturantes `(?:...)`

---

## Passo 5 — Testando a Persistência JSON {#passo-5}

### 5.1 Atualize o Main.java para testar o JsonHandler

Substitua o conteúdo do `Main.java` por:

```java
package tasktracker;

import java.util.List;

public class Main {
    public static void main(String[] args) {

        // ── TESTE 1: Salvar tarefas no arquivo ──────────────────────────
        System.out.println("=== TESTE 1: Salvando tarefas ===");

        List<Task> lista = new java.util.ArrayList<>();
        lista.add(new Task(1, "Comprar leite"));
        lista.add(new Task(2, "Estudar Java"));
        lista.add(new Task(3, "Fazer exercícios"));

        // Altera o status de algumas para testar
        lista.get(1).setStatus("in-progress");
        lista.get(2).setStatus("done");

        JsonHandler.saveTasks(lista);
        System.out.println("✓ Arquivo tasks.json criado/atualizado!");

        // ── TESTE 2: Carregar tarefas do arquivo ────────────────────────
        System.out.println("\n=== TESTE 2: Carregando tarefas ===");

        List<Task> carregadas = JsonHandler.loadTasks();
        System.out.println("Tarefas carregadas: " + carregadas.size());

        for (Task t : carregadas) {
            System.out.println(t); // usa o toString()
        }

        // ── TESTE 3: Verificar se os dados batem ────────────────────────
        System.out.println("\n=== TESTE 3: Verificação de integridade ===");
        for (Task t : carregadas) {
            System.out.printf("ID=%d | Desc='%s' | Status='%s'%n",
                    t.getId(), t.getDescription(), t.getStatus());
        }
    }
}
```

### 5.2 Execute e veja os resultados

Clique em **Run** (Shift+F10).

### 5.3 Resultado esperado

```
=== TESTE 1: Salvando tarefas ===
✓ Arquivo tasks.json criado/atualizado!

=== TESTE 2: Carregando tarefas ===
Tarefas carregadas: 3
[1] Comprar leite                              | Status: todo         | Criado: 2024-01-15 ... | Atualizado: ...
[2] Estudar Java                               | Status: in-progress  | ...
[3] Fazer exercícios                           | Status: done         | ...

=== TESTE 3: Verificação de integridade ===
ID=1 | Desc='Comprar leite' | Status='todo'
ID=2 | Desc='Estudar Java' | Status='in-progress'
ID=3 | Desc='Fazer exercícios' | Status='done'
```

### 5.4 Verifique o arquivo criado

No IntelliJ, na raiz do projeto verá um arquivo `tasks.json`. Clique nele para inspecionar:

```json
[
  {
    "id": 1,
    "description": "Comprar leite",
    "status": "todo",
    "createdAt": "2024-01-15 09:10:00",
    "updatedAt": "2024-01-15 09:10:00"
  },
  {
    "id": 2,
    "description": "Estudar Java",
    "status": "in-progress",
    "createdAt": "2024-01-15 09:10:00",
    "updatedAt": "2024-01-15 09:10:01"
  },
  {
    "id": 3,
    "description": "Fazer exercícios",
    "status": "done",
    "createdAt": "2024-01-15 09:10:00",
    "updatedAt": "2024-01-15 09:10:01"
  }
]
```

> ✅ Se o JSON acima apareceu corretamente e os dados foram carregados de volta, o `JsonHandler` está funcionando!

---

## Passo 6 — Criando a Lógica de Negócio: `TaskManager.java` {#passo-6}

Agora criamos a camada de negócio: a classe que implementa todas as operações do CLI.

### 6.1 Criando o arquivo

1. Clique com o botão direito em `tasktracker` → **New → Java Class**
2. Digite: `TaskManager`
3. Pressione Enter

### 6.2 Digite o código completo

```java
package tasktracker;

import java.util.List;
import java.util.stream.Collectors;

/**
 * Implementa todas as operações de negócio do Task Tracker.
 *
 * Cada método público corresponde a um comando do CLI:
 *   addTask()        → task-cli add
 *   updateTask()     → task-cli update
 *   deleteTask()     → task-cli delete
 *   markInProgress() → task-cli mark-in-progress
 *   markDone()       → task-cli mark-done
 *   listTasks()      → task-cli list
 *
 * Todos os métodos carregam as tarefas do JSON, fazem a operação e salvam de volta.
 */
public class TaskManager {

    // ─── Adicionar ────────────────────────────────────────────────────────────

    /**
     * Adiciona uma nova tarefa com a descrição fornecida.
     *
     * O ID é gerado automaticamente:
     *   - Se não há tarefas → ID começa em 1
     *   - Caso contrário    → ID = maior ID existente + 1
     *
     * Isso garante IDs únicos e crescentes, mesmo após deletar tarefas.
     *
     * @param descricao Texto que descreve a nova tarefa
     */
    public static void addTask(String descricao) {
        // Validação básica: descrição não pode ser vazia
        if (descricao == null || descricao.trim().isEmpty()) {
            System.out.println("Erro: A descrição da tarefa não pode estar vazia.");
            return;
        }

        List<Task> tarefas = JsonHandler.loadTasks();

        // Calcula o próximo ID disponível:
        // mapToInt extrai os IDs, max() pega o maior, orElse(0) retorna 0 se a lista estiver vazia
        int novoId = tarefas.stream()
                .mapToInt(Task::getId)  // Task::getId é uma method reference
                .max()
                .orElse(0) + 1;

        Task novaTarefa = new Task(novoId, descricao.trim());
        tarefas.add(novaTarefa);

        JsonHandler.saveTasks(tarefas);
        System.out.println("Tarefa adicionada com sucesso (ID: " + novoId + ")");
    }

    // ─── Atualizar ────────────────────────────────────────────────────────────

    /**
     * Atualiza a descrição de uma tarefa existente.
     *
     * @param id             ID da tarefa a atualizar
     * @param novaDescricao  Novo texto da descrição
     */
    public static void updateTask(int id, String novaDescricao) {
        // Validação básica
        if (novaDescricao == null || novaDescricao.trim().isEmpty()) {
            System.out.println("Erro: A nova descrição não pode estar vazia.");
            return;
        }

        List<Task> tarefas = JsonHandler.loadTasks();
        Task tarefa = buscarPorId(tarefas, id);

        if (tarefa == null) {
            System.out.println("Erro: Nenhuma tarefa encontrada com ID " + id + ".");
            return;
        }

        tarefa.setDescription(novaDescricao.trim()); // setDescription também atualiza updatedAt
        JsonHandler.saveTasks(tarefas);
        System.out.println("Tarefa " + id + " atualizada com sucesso.");
    }

    // ─── Deletar ──────────────────────────────────────────────────────────────

    /**
     * Remove permanentemente uma tarefa da lista.
     *
     * @param id ID da tarefa a remover
     */
    public static void deleteTask(int id) {
        List<Task> tarefas = JsonHandler.loadTasks();

        /*
         * removeIf() percorre a lista e remove elementos onde o predicado é verdadeiro.
         * Retorna 'true' se pelo menos um elemento foi removido.
         *
         * t -> t.getId() == id  é uma lambda: para cada tarefa t, verifica se o id bate.
         */
        boolean removeu = tarefas.removeIf(t -> t.getId() == id);

        if (!removeu) {
            System.out.println("Erro: Nenhuma tarefa encontrada com ID " + id + ".");
            return;
        }

        JsonHandler.saveTasks(tarefas);
        System.out.println("Tarefa " + id + " removida com sucesso.");
    }

    // ─── Marcar status ────────────────────────────────────────────────────────

    /**
     * Marca uma tarefa como "in-progress" (em andamento).
     *
     * @param id ID da tarefa
     */
    public static void markInProgress(int id) {
        alterarStatus(id, "in-progress");
    }

    /**
     * Marca uma tarefa como "done" (concluída).
     *
     * @param id ID da tarefa
     */
    public static void markDone(int id) {
        alterarStatus(id, "done");
    }

    // ─── Listar ───────────────────────────────────────────────────────────────

    /**
     * Lista tarefas no terminal com formatação visual.
     *
     * @param filtroStatus  Se null/vazio → lista todas.
     *                      Se "todo", "in-progress" ou "done" → filtra por esse status.
     */
    public static void listTasks(String filtroStatus) {
        List<Task> tarefas = JsonHandler.loadTasks();

        // Determina quais tarefas exibir
        List<Task> resultado;

        if (filtroStatus == null || filtroStatus.trim().isEmpty()) {
            // Sem filtro: exibe todas
            resultado = tarefas;
        } else {
            // Valida se o filtro é um status reconhecido
            if (!filtroStatus.equals("todo")
                    && !filtroStatus.equals("in-progress")
                    && !filtroStatus.equals("done")) {
                System.out.println("Erro: Status inválido '" + filtroStatus + "'.");
                System.out.println("      Use: todo | in-progress | done");
                return;
            }

            // Stream filter: seleciona apenas tarefas com o status pedido
            resultado = tarefas.stream()
                    .filter(t -> t.getStatus().equals(filtroStatus))
                    .collect(Collectors.toList());
        }

        // Exibe o resultado
        if (resultado.isEmpty()) {
            if (filtroStatus != null && !filtroStatus.isEmpty()) {
                System.out.println("Nenhuma tarefa com status '" + filtroStatus + "'.");
            } else {
                System.out.println("Nenhuma tarefa cadastrada ainda.");
            }
            return;
        }

        // Cabeçalho visual
        String separador = "─".repeat(110);
        System.out.println(separador);
        System.out.printf("%-5s %-45s %-14s %-21s %-21s%n",
                "ID", "Descrição", "Status", "Criado em", "Atualizado em");
        System.out.println(separador);

        // Uma linha por tarefa
        for (Task t : resultado) {
            System.out.printf("%-5d %-45s %-14s %-21s %-21s%n",
                    t.getId(),
                    t.getDescription(),
                    t.getStatus(),
                    t.getCreatedAt(),
                    t.getUpdatedAt());
        }

        System.out.println(separador);
        System.out.println("Total: " + resultado.size() + " tarefa(s)" +
                (filtroStatus != null && !filtroStatus.isEmpty()
                        ? " com status '" + filtroStatus + "'" : "") + ".");
    }

    // ─── Métodos auxiliares privados ──────────────────────────────────────────

    /**
     * Altera o status de uma tarefa para o valor fornecido.
     * Usado internamente por markInProgress() e markDone().
     *
     * @param id         ID da tarefa
     * @param novoStatus Novo status ("in-progress" ou "done")
     */
    private static void alterarStatus(int id, String novoStatus) {
        List<Task> tarefas = JsonHandler.loadTasks();
        Task tarefa = buscarPorId(tarefas, id);

        if (tarefa == null) {
            System.out.println("Erro: Nenhuma tarefa encontrada com ID " + id + ".");
            return;
        }

        String statusAnterior = tarefa.getStatus();
        tarefa.setStatus(novoStatus); // setStatus também atualiza updatedAt
        JsonHandler.saveTasks(tarefas);
        System.out.println("Tarefa " + id + " atualizada: '" +
                statusAnterior + "' → '" + novoStatus + "'.");
    }

    /**
     * Busca uma tarefa pelo ID dentro de uma lista.
     * Retorna null se nenhuma tarefa com esse ID for encontrada.
     *
     * Usa Stream API:
     *   filter()    → mantém apenas elementos onde getId() == id
     *   findFirst() → pega o primeiro (Optional)
     *   orElse(null) → retorna null se o Optional estiver vazio
     *
     * @param tarefas Lista onde procurar
     * @param id      ID a localizar
     * @return Task encontrado, ou null
     */
    private static Task buscarPorId(List<Task> tarefas, int id) {
        return tarefas.stream()
                .filter(t -> t.getId() == id)
                .findFirst()
                .orElse(null);
    }
}
```

> 💡 **Conceitos aprendidos neste passo:**
> - **Stream API** — `stream()`, `filter()`, `mapToInt()`, `max()`, `findFirst()`, `collect()`
> - **Lambda expressions** — `t -> t.getId() == id`
> - **Method references** — `Task::getId`
> - **`Optional`** — resultado de `findFirst()` / `max()`
> - **`removeIf()`** — remoção condicional de elementos de uma lista
> - **`Collectors.toList()`** — coleta resultado de stream em lista

---

## Passo 7 — Testando o TaskManager {#passo-7}

### 7.1 Atualize Main.java

```java
package tasktracker;

public class Main {
    public static void main(String[] args) {

        // Limpa qualquer tasks.json anterior
        new java.io.File("tasks.json").delete();
        System.out.println(">>> Iniciando com lista vazia\n");

        // ── Teste: add ──────────────────────────────────────────────────
        System.out.println("=== ADD ===");
        TaskManager.addTask("Comprar leite");           // ID: 1
        TaskManager.addTask("Estudar Java Streams");    // ID: 2
        TaskManager.addTask("Fazer exercícios");        // ID: 3
        TaskManager.addTask("Ler documentação Java");   // ID: 4

        // ── Teste: list (todas) ─────────────────────────────────────────
        System.out.println("\n=== LIST (todas) ===");
        TaskManager.listTasks(null);

        // ── Teste: mark-in-progress ─────────────────────────────────────
        System.out.println("\n=== MARK-IN-PROGRESS 2 ===");
        TaskManager.markInProgress(2);

        // ── Teste: mark-done ────────────────────────────────────────────
        System.out.println("\n=== MARK-DONE 1 ===");
        TaskManager.markDone(1);

        // ── Teste: update ───────────────────────────────────────────────
        System.out.println("\n=== UPDATE 3 ===");
        TaskManager.updateTask(3, "Fazer 30 min de exercícios aeróbicos");

        // ── Teste: list por status ──────────────────────────────────────
        System.out.println("\n=== LIST done ===");
        TaskManager.listTasks("done");

        System.out.println("\n=== LIST in-progress ===");
        TaskManager.listTasks("in-progress");

        System.out.println("\n=== LIST todo ===");
        TaskManager.listTasks("todo");

        // ── Teste: delete ───────────────────────────────────────────────
        System.out.println("\n=== DELETE 4 ===");
        TaskManager.deleteTask(4);

        // ── Teste: ID inválido (deve mostrar erro) ──────────────────────
        System.out.println("\n=== DELETE 99 (não existe) ===");
        TaskManager.deleteTask(99);

        // ── Lista final ─────────────────────────────────────────────────
        System.out.println("\n=== LIST FINAL ===");
        TaskManager.listTasks(null);
    }
}
```

### 7.2 Execute e verifique

O output deve mostrar as operações executadas com mensagens de sucesso e, no final, 3 tarefas listadas (a 4 foi deletada).

> ✅ Se tudo funcionou, você já tem 3 das 4 classes prontas!

---

## Passo 8 — Criando o Ponto de Entrada: `Main.java` (versão final) {#passo-8}

Agora sobrescrevemos o `Main.java` de teste pela versão real: o parser de argumentos da linha de comando.

### 8.1 Substitua TODO o conteúdo de Main.java por:

```java
package tasktracker;

/**
 * ╔══════════════════════════════════════════════════════════════════╗
 * ║                    TASK TRACKER CLI — Main                       ║
 * ╚══════════════════════════════════════════════════════════════════╝
 *
 * Ponto de entrada da aplicação.
 * Lê os argumentos da linha de comando e delega para TaskManager.
 *
 * ─── Comandos suportados ──────────────────────────────────────────
 *   task-cli add "descrição"
 *   task-cli update <id> "nova descrição"
 *   task-cli delete <id>
 *   task-cli mark-in-progress <id>
 *   task-cli mark-done <id>
 *   task-cli list
 *   task-cli list todo
 *   task-cli list in-progress
 *   task-cli list done
 */
public class Main {

    public static void main(String[] args) {

        // Se nenhum argumento foi fornecido, exibe a ajuda e sai
        if (args.length == 0) {
            exibirAjuda();
            return;
        }

        // O primeiro argumento é sempre o comando
        String comando = args[0].toLowerCase();

        try {
            switch (comando) {

                // ── add ────────────────────────────────────────────────────────
                case "add":
                    /*
                     * Sintaxe: task-cli add "descrição da tarefa"
                     * args[0] = "add"
                     * args[1] = "descrição da tarefa"  ← obrigatório
                     */
                    if (args.length < 2) {
                        System.out.println("Uso: task-cli add \"descrição da tarefa\"");
                        System.out.println("Exemplo: task-cli add \"Comprar leite\"");
                        return;
                    }
                    TaskManager.addTask(args[1]);
                    break;

                // ── update ─────────────────────────────────────────────────────
                case "update":
                    /*
                     * Sintaxe: task-cli update <id> "nova descrição"
                     * args[0] = "update"
                     * args[1] = "1"              ← ID (número inteiro)
                     * args[2] = "nova descrição"  ← obrigatório
                     */
                    if (args.length < 3) {
                        System.out.println("Uso: task-cli update <id> \"nova descrição\"");
                        System.out.println("Exemplo: task-cli update 1 \"Comprar leite e ovos\"");
                        return;
                    }
                    TaskManager.updateTask(parseInt(args[1]), args[2]);
                    break;

                // ── delete ─────────────────────────────────────────────────────
                case "delete":
                    /*
                     * Sintaxe: task-cli delete <id>
                     * args[0] = "delete"
                     * args[1] = "1"   ← ID (número inteiro)
                     */
                    if (args.length < 2) {
                        System.out.println("Uso: task-cli delete <id>");
                        System.out.println("Exemplo: task-cli delete 1");
                        return;
                    }
                    TaskManager.deleteTask(parseInt(args[1]));
                    break;

                // ── mark-in-progress ───────────────────────────────────────────
                case "mark-in-progress":
                    /*
                     * Sintaxe: task-cli mark-in-progress <id>
                     */
                    if (args.length < 2) {
                        System.out.println("Uso: task-cli mark-in-progress <id>");
                        System.out.println("Exemplo: task-cli mark-in-progress 2");
                        return;
                    }
                    TaskManager.markInProgress(parseInt(args[1]));
                    break;

                // ── mark-done ──────────────────────────────────────────────────
                case "mark-done":
                    /*
                     * Sintaxe: task-cli mark-done <id>
                     */
                    if (args.length < 2) {
                        System.out.println("Uso: task-cli mark-done <id>");
                        System.out.println("Exemplo: task-cli mark-done 3");
                        return;
                    }
                    TaskManager.markDone(parseInt(args[1]));
                    break;

                // ── list ───────────────────────────────────────────────────────
                case "list":
                    /*
                     * Sintaxe: task-cli list [todo|in-progress|done]
                     * args[0] = "list"
                     * args[1] = "todo" ou "in-progress" ou "done"  ← opcional
                     *
                     * Se args[1] não existir, filtroStatus = null → lista todas.
                     */
                    String filtroStatus = (args.length >= 2) ? args[1] : null;
                    TaskManager.listTasks(filtroStatus);
                    break;

                // ── comando desconhecido ───────────────────────────────────────
                default:
                    System.out.println("Comando desconhecido: '" + comando + "'");
                    System.out.println();
                    exibirAjuda();
                    break;
            }

        } catch (IllegalArgumentException e) {
            // Erro de ID inválido (lançado por parseInt() abaixo)
            System.out.println("Erro: " + e.getMessage());
        } catch (Exception e) {
            // Qualquer outro erro inesperado
            System.out.println("Erro inesperado: " + e.getMessage());
        }
    }

    // ─── Métodos auxiliares ───────────────────────────────────────────────────

    /**
     * Converte uma String para int com mensagem de erro amigável.
     *
     * @param texto String a converter (ex: "42")
     * @return Valor inteiro
     * @throws IllegalArgumentException se a string não for um número válido
     */
    private static int parseInt(String texto) {
        try {
            int valor = Integer.parseInt(texto);
            if (valor <= 0) {
                throw new IllegalArgumentException("O ID deve ser um número positivo. Recebido: " + texto);
            }
            return valor;
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException("ID inválido: '" + texto + "'. O ID deve ser um número inteiro.");
        }
    }

    /**
     * Exibe um manual de uso com todos os comandos disponíveis.
     * Chamado quando nenhum argumento é passado ou o comando é desconhecido.
     */
    private static void exibirAjuda() {
        System.out.println();
        System.out.println("╔══════════════════════════════════════════════════════╗");
        System.out.println("║              Task Tracker CLI — Ajuda                ║");
        System.out.println("╚══════════════════════════════════════════════════════╝");
        System.out.println();
        System.out.println("USO:");
        System.out.println("  task-cli <comando> [argumentos]");
        System.out.println();
        System.out.println("COMANDOS:");
        System.out.println("  add \"descrição\"              Adiciona uma nova tarefa");
        System.out.println("  update <id> \"descrição\"      Atualiza a descrição de uma tarefa");
        System.out.println("  delete <id>                  Remove uma tarefa");
        System.out.println("  mark-in-progress <id>        Marca como em andamento");
        System.out.println("  mark-done <id>               Marca como concluída");
        System.out.println("  list                         Lista todas as tarefas");
        System.out.println("  list todo                    Lista tarefas pendentes");
        System.out.println("  list in-progress             Lista tarefas em andamento");
        System.out.println("  list done                    Lista tarefas concluídas");
        System.out.println();
        System.out.println("EXEMPLOS:");
        System.out.println("  task-cli add \"Comprar leite\"");
        System.out.println("  task-cli update 1 \"Comprar leite e ovos\"");
        System.out.println("  task-cli mark-in-progress 2");
        System.out.println("  task-cli list done");
        System.out.println("  task-cli delete 3");
        System.out.println();
    }
}
```

---

## Passo 9 — Testes Completos via Terminal {#passo-9}

Antes de gerar o JAR, vamos testar pelo IntelliJ passando argumentos.

### 9.1 Configurando o Run com argumentos no IntelliJ

1. No canto superior direito, clique em **"Main"** → **Edit Configurations...**
2. No campo **Program arguments**, digite: `add "Comprar leite"`
3. Clique **OK** → clique **Run**
4. Repita para cada comando abaixo

### 9.2 Sequência de testes completa

Execute cada um na ordem, alterando o campo **Program arguments**:

| # | Program arguments | Resultado esperado |
|---|-------------------|--------------------|
| 1 | `add "Comprar leite"` | `Tarefa adicionada com sucesso (ID: 1)` |
| 2 | `add "Estudar Java"` | `Tarefa adicionada com sucesso (ID: 2)` |
| 3 | `add "Fazer pull-ups"` | `Tarefa adicionada com sucesso (ID: 3)` |
| 4 | `list` | Exibe as 3 tarefas com status `todo` |
| 5 | `mark-in-progress 2` | `Tarefa 2 atualizada: 'todo' → 'in-progress'` |
| 6 | `mark-done 1` | `Tarefa 1 atualizada: 'todo' → 'done'` |
| 7 | `list done` | Exibe apenas tarefa 1 |
| 8 | `list in-progress` | Exibe apenas tarefa 2 |
| 9 | `list todo` | Exibe apenas tarefa 3 |
| 10 | `update 3 "Fazer 30 pull-ups e 50 push-ups"` | `Tarefa 3 atualizada com sucesso.` |
| 11 | `delete 1` | `Tarefa 1 removida com sucesso.` |
| 12 | `delete 99` | `Erro: Nenhuma tarefa encontrada com ID 99.` |
| 13 | `add ""` | Mensagem de erro: descrição vazia |
| 14 | `list` | Exibe tarefas 2 e 3 com os status atualizados |

> ✅ Se todos os testes passaram, sua aplicação está pronta!

---

## Passo 10 — Gerando o JAR Executável {#passo-10}

Para executar como `task-cli` no terminal (igual ao desafio), precisamos empacotar em um `.jar`.

### 10.1 Criando o Artifact no IntelliJ

1. Menu **File → Project Structure** (Ctrl+Alt+Shift+S)
2. Na aba esquerda, clique em **Artifacts**
3. Clique no **+** → **JAR → From modules with dependencies**
4. Configure:
   - **Main Class:** clique em `...` e selecione `tasktracker.Main`
   - Clique **OK**
5. Clique **OK** novamente para fechar o Project Structure

### 10.2 Build do JAR

1. Menu **Build → Build Artifacts...**
2. Selecione `task-tracker:jar`
3. Clique **Build**

O arquivo será gerado em:
```
task-tracker/out/artifacts/task_tracker_jar/task-tracker.jar
```

### 10.3 Testando pelo terminal do sistema

Abra o terminal do seu sistema operacional (não o do IntelliJ) e navegue até a pasta onde está o JAR:

```bash
cd /caminho/para/task-tracker/out/artifacts/task_tracker_jar/
```

Execute os comandos:

```bash
# Adicionar tarefas
java -jar task-tracker.jar add "Comprar leite"
java -jar task-tracker.jar add "Estudar Java"
java -jar task-tracker.jar add "Fazer exercícios"

# Listar todas
java -jar task-tracker.jar list

# Mudar status
java -jar task-tracker.jar mark-in-progress 2
java -jar task-tracker.jar mark-done 1

# Listar por status
java -jar task-tracker.jar list done
java -jar task-tracker.jar list todo

# Atualizar
java -jar task-tracker.jar update 3 "Fazer 30 minutos de corrida"

# Deletar
java -jar task-tracker.jar delete 2

# Lista final
java -jar task-tracker.jar list
```

### 10.4 Criando o alias `task-cli` (opcional)

**Linux/macOS:**
```bash
# Adicione ao seu ~/.bashrc ou ~/.zshrc:
alias task-cli='java -jar /caminho/completo/task-tracker.jar'

# Depois execute:
source ~/.bashrc

# Agora você pode usar:
task-cli add "Minha tarefa"
task-cli list
```

**Windows (PowerShell):**
```powershell
# Adicione ao seu perfil PowerShell:
function task-cli { java -jar "C:\caminho\completo\task-tracker.jar" @args }
```

---

## Finalizando

Você construiu um CLI completo em Java puro que:

- ✅ Aceita argumentos posicionais pela linha de comando
- ✅ Persiste dados em arquivo JSON sem bibliotecas externas
- ✅ Implementa todas as operações: add, update, delete, mark-in-progress, mark-done, list
- ✅ Trata erros graciosamente (ID inválido, tarefa não encontrada, etc.)
- ✅ Mantém timestamps de criação e atualização

### Estrutura final do projeto

```
task-tracker/
├── src/
│   └── tasktracker/
│       ├── Task.java           ← Modelo de dados
│       ├── JsonHandler.java    ← Persistência JSON
│       ├── TaskManager.java    ← Lógica de negócio
│       └── Main.java           ← CLI / ponto de entrada
├── out/
│   └── artifacts/
│       └── task_tracker_jar/
│           └── task-tracker.jar
└── tasks.json                  ← Criado automaticamente na 1ª execução
```

### Próximos desafios (para continuar aprendendo Java)

- Adicionar **prioridade** às tarefas (low / medium / high)
- Implementar **busca** por palavra-chave na descrição
- Adicionar **data de vencimento** (due date)
- Refatorar para usar **Maven** e adicionar testes com **JUnit**
- Exportar relatório em **CSV** ou **HTML**